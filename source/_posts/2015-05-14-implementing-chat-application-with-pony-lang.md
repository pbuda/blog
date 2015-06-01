title: "Implementing chat application with Pony lang"
date: 2015-05-14
tags:
- pony lang
- actors
- capabilities
- concurrency
categories:
- Software Development
- Pony
---

Recently on Hacker News I spotted a post on a new object-oriented, actor-model programming language called [Pony](http://ponylang.org/). Given it is pretty new (as of this post the current version is 0.1.3) I decided to give it a try and hack a little.

I've went through a tutorial that is available at [Pony's site](http://tutorial.ponylang.org/) and found the language to be pretty interesting, so I decided to implement my first simple application - a chat. I think it will allow me to experiment with a few language features and will be enough for a start with the new language. I'd like to share with you what I created and give you some introduction to the language. I will also try to explain a few things on the way but please remember that explaining everything in detail is impossible in one post, hence more will come :)

The architecture of this application will be pretty simple. We'll have a server, that will manage user sessions and a client that will login to the server and post messages to it. After each message is posted the chat log will be displayed on the screen. For starters the whole application will be hardcoded, without remote calls and such - time will come to implement that too :) (I don't think there are remote actors in Pony yet!)

<!-- more -->

### Client

The client is a simple class that encapsulates server calls. It exposes just three methods: `login`, `post` and `logout`. This is how it looks like:

``` pony
class Client 
  let username:String
  let server:Server

  new create(username':String, server':Server) =>
    username = username'
    server = server'

  fun login() => server.login(username)

  fun logout() => server.logout(username)

  fun post(message:String) => server.post(username, username + ": " + message)
```

Let's talk about what we can see here.

Classes in Pony are defined using the `class` keyword. Classes can have fields defined with either `var` or `let` and methods defined with `fun`. Fields defined with `var` keyword can change in time - you can set it's default value in constructor and then reassign it sometime later, while fields defined with `let` are instantiated once (either in constructor or inline in declaration) and then can't be reassigned later on. 	

In Pony classes and actors can have many constructors, and unlike Java, they don't have to be named like the class they are part of. The constructor is marked with the `new` keyword. The default constructor name is `create` (which is important for some sugar to work) but you can have any name here you'd like. Think of it as a factory method so you can instantiate your class or actor in a specific way with constructor name really describing your intent. Quite neat.

You can spot a simple convenience stuff in the argument list of the constructor. The `username'` and `server'` arguments are followed by the `'` because you can't have same names in a scope, you also can't reference fields with `this`. So Pony allows you to create a `prime` of the argument so you don't have to provide different names.

Then come the definitions of methods which are defined using the `fun` keyword. Methods also have to have parenthesis, even empty ones. In this example methods don't return any values, they simply delegate to another object.

### Server

The server is responsible for managing user sessions and storing the chat log in some persistent storage. Because of that, the server is split into a few smaller parts. The topmost part is the `Server` actor that the `Client` is calling. It stores user sessions and delegate chat messages to the storage. The `trait Storage` is used to define an interface for the storage system and a simple `actor MemoryStorage` implements this trait. In order to retrieve the messages, an `interface LogPrinter` is introduced which defines a method to print a message.

Let's start with implementing the storage.

``` pony
interface LogPrinter 
  be print(message:String)

trait Storage tag
  be push(message:String)

  be print(printer:LogPrinter tag)
```

There's quite a few new things in this snippet to notice. Both defined types are `interface LogPrinter` and `trait Storage tag`. Pony has two types of subtyping: structural (using `interface`) and nominal (using `trait`). Structural subtyping is a kind of subtyping where a name doesn't matter and only how a type is built. On the other hand nominal subtyping also checks names. Another thing to notice is the `tag` keyword. This is a `capability`. Pony introduces a concept of `capabilities` to make guarantees about sharing objects between actors so that compiler can check that. This is important for safe concurency where Pony tries to excell. One last thing to note is that methods in both types are defined using `be` keyword. This keyword denotes a behaviour on an actor. We'll talk about it in a second.

##### Storage

First let's see how `trait` is used and later we'll see how `interface` is used.

``` pony
actor MemoryStorage is Storage 
  let _env:Env
  
  let log:List[String]

  new create(env:Env) =>
    _env = env
    log = List[String]

  be push(message:String) => 
    _env.out.print("Pushing message: " + message)
    log.push(message)

  be print(printer:LogPrinter tag) =>
    _env.out.print("Printing log")
    try
      for message in log.values() do
        printer.print(message)
      end
    end
```

Here we implement the `trait Storage`, this is achieved by specifying the list of traits after the keyword `is`. To implement a `trait` we need to define implementation of the methods specified in `trait Storage`. We define a variable `let _env:Env` - `Env` is a class representing the environment, it holds references to input and output and program arguments. Our `actor MemoryStorage` will store the messages as a simple list of strings, so we define a `let log:List[String]` variable. I won't go into detail of how `List[A]` works, so if you're interested in Pony collections, you can browse the code at [Pony's GitHub](https://github.com/CausalityLtd/ponyc/tree/master/packages/collections).

Now let's get back to the `be` keyword. Pony actors expose their logic via behaviours. Behaviours differ to functions in that behaviours are asynchronous. You can't specify a return type for a behaviour, because it always returns the receiver (the actor on which the behaviour is called) so that you can chain multiple behaviours. A note to remember is that even though actors allow concurrent programming, execution of code in actor is sequential, so you don't have to worry about concurrency while writing actor code.

I'd like to point out one thing I don't like in this piece of code. The `trait Storage` defines methods as behaviours, because you can't have a `trait` or an `interface` with `fun`s and then implement that in actor as `be`s and the other way around. It seems then that looking at `trait` or `interface` definition you already know whether you'll be working with actor or an object. In a way it's fine, because that makes you realize you'll be working with async code, on the other hand it doesn't seem flexible. I guess there might be something deeper in this and I will have to investigate it further.

##### Session

Next piece of the application is the session management actor. It's pretty simple.

``` pony
actor Session
  let username:String
  let storage:Storage

  new create(username':String, storage':Storage) =>
    username = username'
    storage = storage'

  be post(message:String) =>
    storage.push(message)
```

This session manager acts as a bridge between the server and the storage. One thing to note here is that behaviour `be post(message:String)` delegates to the storage. In such case the caller of the `post` method will get the `Session` actor back as it's the direct receiver. I'm not sure it's possible to `forward` behaviours as in for example Akka, or maybe it just isn't implemented yet (remember, I was working with version 0.1.2!)

##### Actual server

And finally we have an `actor Server` that orchestrates the whole thing.

``` pony
actor Server 
  let storage:Storage
  let sessions:Map[String, Session]
  let env:Env

  new create(env':Env) =>
    env = env'
    storage = MemoryStorage(env)
    sessions = Map[String, Session]

  be login(username:String) =>
    try
      sessions.insert(username, Session(username, storage))
    else
      env.out.print("Error creating session for " + username)
    end

  be logout(username:String) =>
    try
      sessions.remove(username)
    end

  be post(from:String, message:String) =>
    try
      sessions(from).post(message)
    else
      env.out.print("Could not find session for " + from)
    end

  be print_log() =>
    storage.print(this)

  be print(message:String) =>
    env.out.print(message)
```

Again, a few new concepts kick in. Let's see how the `Session` actor is created in the `be login(username:String)` method. The normal way of calling constructors is by invoking `ClassName.create()` or any other constructor name. Pony provides some sugar for you and here, the default constructor `create` can be called implicitly. So the call to `Session(username, storage)` is actually `Session.create(username, storage)`.

But wait, there's more sugar! Take notice of how the session is retrieved in `be post(from:String, message:String)` method. The `Map` defines a special method `fun apply(key: box->K!): this->V ?` and Pony allows you to call that method directly on an object. So the call `sessions(from)` is actually `sessions.apply(from)`. This is pretty useful as you can define `apply` method for your classes that would be their default (or convenient) action.

One last thing to talk about here are the `be print_log()` and `be print(message:String)` methods. The latter has been defined in the `interface LogPrinter` but as I've mentioned before, interfaces are used for structural subtyping, so no naming is required. That is why `actor Server` does not define `is LogPrinter` (although it could, for readability) but it actually is one, because it defines the behaviour presented in that `interface`. That's why we can call the `MemoryStorage.print(LogPrinter)` method using `this` keyword.

### Running the application

Compiling Pony code is pretty easy - just run `ponyc` (assuming you've got it installed) in the folder of your program and you will get a few artifacts out of it, one being an executable to run. But to run a Pony program you need an actor that is called Main. So let's add it and run our program.

``` pony
actor Main
  new create(env:Env) =>
    let server = Server(env)
    let c1 = Client("client1", server)
    let c2 = Client("client2", server)
    c1.login()
    c2.login()
    c1.post("Hi!")
    server.print_log()
    c2.post("Hi!")
    server.print_log()
    c1.logout()
    c2.logout()
```

Pretty simple. But it doesn't work as expected. Since behaviours as asynchronous, the output of the program is indefinite. Sometimes you'll get just the first message sent, sometimes all. I'm not sure why so maybe some more skilled reader will suggest a solution :)

### Summary

So that's it folks! I've had quite fun time implementing that, reading about and using this new kid on the block. Did I like Pony? Yes, given it's very new and a lot of stuff is not implemented yet, it gives hope. Where would I use Pony? Don't know yet, it's actor-model based and has some guarantees about concurrency, so it might work very well in networking applications and highly concurent computation stuff. I definitely will follow the development of the language and also write some more blog posts about it's features. There's certainly a lot to be covered, since the documentation is not yet fully developed and some parts (like capabilities) are hard to understand. The full code of this chat application is available at [GitHub]()