title: "Pony lang capabilities - understanding the defaults"
date: 2015-06-30 13:18:20
tags:
- pony
- capabilities
- defaults
- concurrency
categories:
- Software Development
- Pony
---


Pony offers concurrency programming using actors. However safe concurrency is hard. In order to make it safe, you have to remember about a few things. First of all, passing mutable data is hard, because concurrent reading and writing to that data can lead to garbage being store, or overwritten state. So concurrent programs have to employ safety mechanisms (such as locks, semaphores) to work with mutable data. However it's error prone and introduces overhead, not to mention performance impact. [Pony tutorial](http://tutorial.ponylang.org/capabilities/introduction/) on capabilities mentions two ways of sharing data between actors - sharing immutable data and passing isolated data.

Sharing immutable data is safe, because once created this data cannot be modified. Think of sharing as moving reference to data around, leaving a trace of it everywhere you want. Isolated data (data for which only one reference exists) is safe, because only one actor can have reference to that data and changing mutable data in one actor is perfectly ok. So passing means that you pass the reference to someone else and don't store it anymore.

Pony compiler can perform checks whether your program only shares immutable data or is passing isolated data. To achieve that, Pony introduces a mechanism called capabilities which makes some guarantees about sharing objects with other actors. These guarantees then allow the Pony compiler to check for errors in your code and whether your concurrent app will be safe. At first I found this topic to be confusing so I decided to write a post about it and maybe you, the reader, will have easier time understanding this concept.

<!-- more -->

### Pony Capabilities

Pony capabilities are a kind of qualifier with which you can describe a way the language construct will be used in a concurrent program. There are six capabilities to choose from and some of them are for immutable data, some for isolation and one opaque that prevents object from being accessed.

- `ref` (Reference) is your normal, mutable data. This data can be freely modified and read by a single actor or other variables, but it can't be shared with other actors.
- `val` (Value) is for immutable data, which means that no-one can modify it, so it's safe to read and share that data.
- `iso` (Isolated) marks a variable as being isolated so there's no other variable that has access to that data, therefore it's safe for your actor to read and write to it.
- `box` (Box) means that this data is read-only to you. There might be other variables writing to that data, but you can only read it.
- `trn` (Transition) allows you to use variable as a write-only.
- `tag` (Tag) is used for identification. You can neither write nor read from that variable, but you can use it to identify objects or share it with other actors.

Some language constructs specify default capabilities that make sense in most cases. Below we will explore what these defaults are.

### Actor default

By default all actors are `tag`s and you can't change it - compiler will complain about such attempt:
```pony
actor Act val
  be dosmth() => None
```
```
/Volumes/Personal/pony/capabilities/main.pony:12:11: actor cannot specify default capability
actor Act val
```

By being a `tag` actors can be safely passed between your objects and you can call behaviors on them.
```pony
actor Main
  new create(env:Env) =>
  var foo:Foo = Foo
  let a = Act
  foo.dosmth(a)

class Foo
  new create() => None
  fun dosmth(a:Act) =>
    a.dosmth()

actor Act
  be dosmth() => None
```

### Primitive default

Primitives are always `val`s and similar to actors you cannot specify a capability for primitives.

### Class default

When you define a new `class` by default it's a `ref`. This means that when you create a new instance of this class it will default to `ref` but you can override that by defining a capability on variable (more on that later).

You can however specify the default capability your new instances will default to:
```pony
actor Main
  new create(env:Env) =>
  var foo:Foo = Foo

class Foo box
  var a:Array[U8] = Array[U8]
  new create() => None
```

Variable `foo` in this case is a `box`. That is because you specified a default capability in the class definition. If you now try to modify the `a` variable compiler will complain about unsafe operation
```pony
actor Main
  new create(env:Env) =>
  var foo:Foo = Foo
  foo.a = Array[U8]

class Foo box
  var a:Array[U8] = Array[U8]
  new create() => None
```
```
/Volumes/Personal/pony/capabilities/main.pony:4:9: not safe to write right side to left side
  foo.a = Array[U8]
```

### Constructor default

Constructors in Pony also have a default return type, which is `ref`. It's easily changeable though, by specifying the capability name right after the `new` keyword.
```pony
class Foo val
  new val create() => None
```

It is important to remember that while specifying the class default makes your defined variable use the same capability as class, it might be incompatible with the constructor capability.
```pony
actor Main
  new create(env:Env) =>
  var foov:Foo = Foo

class Foo val
  new create() => None
```
```
/Volumes/Personal/pony/capabilities/main.pony:3:16: right side must be a subtype of left side
  var foov:Foo = Foo
               ^
/Volumes/Personal/pony/capabilities/main.pony:6:3: right side type: Foo ref
  new create() => None
  ^
/Volumes/Personal/pony/capabilities/main.pony:3:12: left side type: Foo val
  var foov:Foo = Foo
```

One of possible solutions is to specify the returning capability for `Foo`s constructor.
```pony
class Foo val
  new val create() => None
```

Another possibility is to use the `recover` block but this is out of scope of this article.
```pony
actor Main
  new create(env:Env) =>
  var foov:Foo = recover val Foo end

class Foo val
  new create() => None
```

### Function default

Functions can have capabilities defined in two places in their declaration. The first place is function's return type
capability. This uses class defaults but can be overridden by specifying a concrete capability.
```pony
actor Main
  new create(env:Env) =>
    let f = Foo

class Foo
  let a:String = "a"

  fun get_a():String iso => a.clone()
```

`String` is a `val` by default but here we define in our function that we want the resulting type to be actually `String iso`.

The other place you can set the capability on a function declaration is right after the `fun` keyword. This capability will define
how the function can behave. By default this capability is `box` which means that data in the owner class is read only to that method.
This plays well with `box` definition - there are other places the variables can change (other methods that can change the class fields)
but you can only read them. Trying to modify fields in a default `box` function will make compiler complain
```pony
class Foo
  var a:String = "a"

  fun get_a():String =>  a = a + "b"
```
```
/Volumes/Personal/pony/capabilities/main.pony:4:28: cannot write to a field in a box function
  fun get_a():String =>  a = a + "b"
                           ^
```

But change that to something that is write capable (`iso`, `trn`, `ref`) and you can modify your class contents again
```pony
class Foo
  var a:String = "a"

  fun ref get_a():String =>  a = a + "b"
```

### Variable default

Variables default to whatever they are created for. When instantiating a class, then it will default to `ref` or
if class has a default set, to whatever was set in class definition. For actors it will always be `tag` because actors
are always `tag`s. For primitives it will be `val`.

As mentioned in the section about classes, variables will default to capability specified in class (or `ref` if none was specified). This can be changed
by providing capability name right after type declaration
```pony
actor Main
  new create(env:Env) =>
  var foo:Foo iso = recover Foo end

class Foo val
```

For function return types however, if you don't use type inference (you define type of your variable explicitly) then you have to provide the capability that can be used
as alias capability (aliasing is out of scope of this article). In the example below we use `box` as the variable capability but we could also use `val` or `tag`. This is because
as mentioned before, variables default to whatever is the default of their type.
```pony
actor Main
  new create(env:Env) =>
  var foo:Foo = Foo
  var s:Seq[String] box = foo.get()

class Foo
  fun get():Seq[String] val => recover Array[String] end
```

### Summary

Pony capabilities are used throughout the Pony code so some sensible defaults need to be set. Knowledge of these can help you understand compiler warnings and adapt your code to conform to capability rules.
