title: "Pony lang capabilities unchained (with examples)"
tags:
- pony lang
- capabilities
- pony guarantees
- concurrency
---

Pony offers concurrency programming using actors. However safe concurrency is hard. In order to make it safe, you have to remember about a few things. First of all, passing mutable data is hard, because concurrent reading and writing to that data can lead to garbage being store, or overwritten state. So concurrent programs have to employ safety mechanisms (such as locks, semaphores) to work with mutable data. However it's error prone and introduces overhead, not to mention performance impact. [Pony tutorial](http://tutorial.ponylang.org/capabilities/introduction/) on capabilities mentions two ways of sharing data between actors - sharing immutable data and passing isolated data.

Sharing immutable data is safe, because once created this data cannot be modified. Think of sharing as moving reference to data around, leaving a trace of it everywhere you want. Isolated data (data for which only one reference exists) is safe, because only one actor can have reference to that data and changing mutable data in one actor is perfectly ok. So passing means that you pass the reference to someone else and don't store it anymore.

Pony compiler can perform checks whether your program only shares immutable data or is passing isolated data. To achieve that, Pony introduces a mechanism called capabilities which makes some guarantees about sharing objects with other actors. These guarantees then allow the Pony compiler to check for errors in your code and whether your concurrent app will be safe. At first I found this topic to be confusing so I decided to write a post about it and maybe you, the reader, will have easier time understanding this concept.

<!-- more -->

### Pony Capabilities

Pony capabilities are a kind of qualifier with which you can mark your variables (not types!) to describe a way the variable will be used in a concurrent program. There are six capabilities to choose from and some of them are for immutable data, some for isolation and one opaque that prevents object from being accessed.

- `ref` (Reference) is your normal, mutable data. This data can be freely modified and read by a single actor or other variables, but it can't be shared with other actors.
- `val` (Value) is for immutable data, which means that noone can modify it, so it's safe to read and share that data.
- `iso` (Isolated) marks a variable as being isolated so there's no other variable that has access to that data, therefore it's safe for your actor to read and write to it.
- `box` (Box) means that this data is read-only to you. There might be other variables writing to that data, but you can only read it.
- `trn` (Transition) allows you to use variable as a write-only.
- `tag` (Tag) is used for identification. You can neither write nor read from that variable, but you can use it to identify objects or share it with other actors.

By default, all `class`es are `ref`s and all `actor`s are `tag`s. Variables are also 'ref's and functions return `box`es. Behaviours always return the receiver as `tag`.

### Defining capabilities

You can define capabilities in several places with fields and variables being the most common. But there are other places too. While `actor`s can never specify the default capability you can change that for `class`es.
```pony
class Foo val
```
The above definition states that all occurences of `class Foo` will always be `val`s so you will not be able to do
```pony
let foo:Foo = Foo
```
because the default capability for `Foo` is `val` and Foo implicit constructor will return `ref`. However Pony implements type inference and this notation would work
```pony
let foo = Foo
```
as this notation will create a `foo:Foo ref` variable. Also this is just the default, so you can easily override it at the variable declaration time
```pony
let foo:Foo ref = Foo
```
Functions in Pony return `box`s by default, but you can change that by providing capability name between `fun` and function's name, like
```pony
fun iso produceData():Data
```
This function will now return `iso` results.

Constructors by default return `ref`s but you can also specify a capability to return by constructor
```pony
class User
  let username:String
  new iso create(username':String) =>
    username = username'
```
This might be helpful if you know that you will always use this class with certain capability, or providing a secondary constructor for most common use cases.

### Aliasing



### Ref

### Val

### Iso

### Box

### Trn

### Tag
