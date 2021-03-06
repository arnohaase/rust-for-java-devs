# A Java Devoloper's guide to Rust

This is a list of stumbling blocks that I came across when I started doing Rust. 
I come from a Java background, so naturally that influenced my implicit expectations
and assumptions. This document may or may not be helpful for people coming from 
other backgrounds - I wouldn't know :-).

This document is just a list of insights that were not obvious for me. It is not
intended as a comprehensive overview of the language or a tutorial. For that, there is
[The Book](https://doc.rust-lang.org/book) and other excellent resources on the
[Rust Homepage](https://rust-lang.org/learn) and elsewhere.

I suggest you read this document before you start a tutorial, and come back to it
when you feel the need.

Anyway, here goes.

## General observations

### Same name, different concept

There are things in Rust that have familiar sounding names but are just different
enough to be confusing. You'd best forget that Java has things that are named the
same, and learn the Rust features from the ground up. Most importantly, try **not**
to use proven Java idioms with Rust features of the same name - they probably 
won't work or lead to pretty un-idiomatic Rust code.

* Arrays (must have a length known at compile time)
* Enums (something really different from Java enums)
* Traits (look like Java interfaces but aren't really) 

There are detailed sections for these below.

### Compile time checking

Rust is built around the concept of checking stuff at compile time. This can get
in the way. Especially while you get started, this will probably often feel like 
"Leave me alone already, I know what I'm doing!". Well, that's intentional, and
stringent compile time checks are at the heart of Rust's language design. 

If you write something you think is correct but the compiler disagrees, that 
is often a sign of a subtle bug. So learn to see compiler checks as your friend
and not a necessary evil. 

Doing this will require you to unlearn some Java best practices and learn
idiomatic ways to work with Rust and allow the compiler to do its checks.
Acknowledge and embrace this - if you don't want that, Rust probably isn't the 
language for you.

### Standard library

When I made my first steps with Rust, I tried to do things with 'just the language'
without the standard library. That's what I usually like to do when learning a new
language, I like to understand the language on its own, with its specific abstractions
and sugar.

Well, that didn't work for Rust, at least not the way I tried it. There are parts
of the standard library that are necessary to do the most basic stuff:

* `Vec` for dynamically-sized memory
* `Box` (and pretty soon `Rc` / `Arc`) to get stuff onto the heap

In hindsight, that is not a problem. It's just that other modern languages like
Java (or C++, for that matter) support these things as part of their core 
language, and I struggled for a while trying to do these things "directly".

My point is: While `Vec` and `Box` are part of the library, you will really
really need them to take your first steps with Rust. There is no easy way
to work with pointers or dynamically allocated lists without using "just
language features". 

Embrace them and treat them as if they were part of the language core, at least until you know 
enough to understand how they are implemented.

## Features

### Arrays

Arrays in Rust are something completely different from JVM arrays - try to ignore
that they have the same name. 

Rust arrays are a continuous area of memory with a **size known at compile time**.
An array of two bytes `[u8;2]` and an array of three bytes `[u8;3]` are totally
different data types. They can not be assigned to each other, there is no common
supertype - there is no way to use them interchangeably.

That means there is **no way to create an array with a size known at runtime**. If
you need something like that, using `Vec` from the standard library is the way to 
go - this is one more example that dynamically sized stuff is 'special' in Rust
and less simple to do than statically sized stuff. 
 
And finally, arrays do **not** involve a pointer. An array can be allocated on the
stack, and if an array is part of a struct, it is part of that struct's flat memory
layout. Like any other data type, an array must reside behind a pointer in order
to live on the heap.

### Slices

Rust has *slices*, which are pointers to an array or part of it. They are
"fat" pointers that contain length information which is checked at runtime and
not part of the slice type. 

Slices have a type signature similar to arrays: `[u8;2]` is a byte array
while `[u8]` is a slice of bytes. Though they look similar in code, they are
very different beasts:

* A slice **never owns** data. It is just a (borrowed) **view** to data owned
e.g. by an array.
* There is no way to allocate a slice directly. Slices are **not** a way to 
dynamically allocate memory. 

Slices are part of Rust's language core which may take some getting used to
coming from Java.

### Enums and polymorphism

Rust enums are designed so they can hold state, and **different state in different
instances**. The following enum represents an IP address which can be either 
IP4 or IP6:
 
```rust
enum InetAddress {
  V4(u8, u8, u8, u8),
  V6(String)
}
``` 

This is completely different from Java: `V4` has different **instances** holding
actual IP addresses. 

In Rust, this is a solution for many problems where you would use inheritance
in Java. A Rust `enum` has a fixed number of variants that can each have different
kinds of data, and functions can pattern match (Rust's more powerful equivalent
of Java's `switch`) to handle different variants.  

Technically, this is not polymorphism but 
[Algebraic Types](https://en.wikipedia.org/wiki/Algebraic_data_type). But if
feel like "I'd like some polymorphism here", you should start by trying to
model that using enums. At least while you get started, so you get a feel
for this language feature which Java does not have.

If you are like me, this might raise a lot of concerns and leave you wondering
"how do I do X". 

Well, firstly `trait`s do exist in Rust, it's just that they are not your first
choice. But secondly, while I don't have all the answers yet, using enums for
polymorphism works pretty well once I got used to it. Try to be open minded. 

### Heap

If you want to put stuff onto the heap, you one of the pointer structs (unless
you go reeeeally low level using unsafe Rust - but if you know how to do that, 
you probably don't need this document any more). 

For your first steps, that means `Box` or `Arc`. 

`Arc` is roughly the equivalent of a Java object reference: You can have many
`Arc` instances pointing to the same object in memory, and when the last of 
those is gone, the object is cleaned up ("dropped" in Rust terminology). 
When you need another reference to the same object, you `clone()` it. (Btw, 
"Arc" stands for "Atomic Reference Counter".)

Many Rust tutorials start with `Box` as the first kind of pointer. A `Box` is
the **single** reference to some object, and when the `Box` goes out of scope,
the object is dropped. 

This is conceptually simpler than `Arc` and helps explain Rust's lifecycle 
concepts. `Box` is however very different from what you are used to in Java.

The take-away is this: `Box` is great for learning lifecycle concepts, but if
you hit a wall trying to do stuff that "should be simple", try `Arc` instead.
And don't worry, this will feel natural pretty soon.

### Design for ownership and mutability

One of the Rust's core rules basically says that **mutation requires exclusive 
access**. This is of course a simplification, and there are ways around this
(`Cell`, atomic data types, ...), but the rule is pretty fundamental and is
omnipresent in practice. 

It boils down to "shared state is read-only". You can work around this e.g. by
adding a `Mutex` as a level of indirection and keep doing Java-style design, 
but that will likely feel weird and complex. 

In idiomatic Rust, you plan your abstractions and data structures around 
ownership and mutability. It may lead to splits between mutable and immutable
data that you would combine into a single Java class.  

Question your Java-based intuition and experience for creating abstractions,
and embrace Rust design as a new paradigm. That will probably feel weird at 
first (it did for me), but it will likely lead to new insights as well once
you embrace it.   

### Traits, Fat Pointers and Dynamically Sized Types

In memory, a Rust `struct` consists only of its fields. There is no run-time
type information that would allow reflection of any kind, there is no 
method table that would allow polymorphic invocation of method, there is
nothing beyond the struct's raw data. 

For `enum`s, the compiler stores an additional byte or so to determine which
of the enum's variants the data represents. But even for enums, Rust stores
just enough to determine a given enum's variant and no more.  

This works great when the compiler knows the data's concrete type. Which is
the normal case in Rust.

Obviously, this does not allow any "real" polymorphism however, where the caller 
only knows that a given objects implements a certain `trait` and calls
one of the trait's methods.

#### Polymorphism via pointers

Rust does allow pointers with a `trait` as their static type. And you can 
invoke methods on these pointers. There is e.g. the `FnOnce` trait which
is implemented by all closures:

```rust
let f = || { println!("Hi"); };
let fn_ptr: Box<FnOnce()> = Box::new(f);
```

Rust does this by using a trick under the hood: a pointer to a trait is stored
as two pointers internally, one of them to the object and one to a virtual
method table. These pointers are called "fat" pointers, and you should usually
not have to worry about them.

#### Dynamically Sized Types

That is, as long as you work with them through pointers. When you start passing
objects around "directly" via a `trait` or have a field in a struct that has 
a `trait` as its type, you will see compiler errors saying that the type's size
is not known at compile time. 

That is because Rust can not know at compile time how much memory to allocate
for data based on a trait - and for many things the compiler needs to do just
that.

#### The take-away

Rust sometimes exposes you as a programmer to memory layout issues. Rust
focuses on doing stuff at compile time, therefore some stuff requires data
size to be known at compile time.
