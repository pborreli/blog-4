---
title: "Erlang is the Most Object Oriented Language"
date: 2013-10-03T11:49:00+01:00
---

Here's an idea: [Erlang](http://www.erlang.org/) is the only true object oriented
language in use today.

You might be thinking "[WATTTT](http://i.imgur.com/lH5ExZq.gif), that doesn't
make any sense". But bear with me. Let's examine this idea a bit and see where
it takes us.

## Functional Programming and Object Orientation
Often when you ask programmers what the opposite of functional programming is,
they simply reply "well, object oriented programming, of course!".

Unfortunately, this isn't true. Object orientation and functional programming are
[orthogonal to one another](http://stackoverflow.com/questions/3949618/are-fp-and-oo-orthogonal),
meaning, in more layman's terms, that you can have your cake and eat it too. A language
can be both object oriented and functional. While most functional
languages do not implement an object system, [Scala](http://www.scala-lang.org/)
and [F#](http://www.tryfsharp.org/) are both popular languages that attempt to combine
the two paradigms.

Functional programming at its core cares about concepts such as immutability and
first class functions which are not incompatible with object orientation.

So Erlang is not excluded from the realm of object orientated languages simply due to
its functional nature. In order to see how Erlang fits into the world of object
orientation we need to examine what exactly OO is.

## What is Object Orientation?

Wikipedia's [explanation of object orientation](http://en.wikipedia.org/wiki/Object-oriented_programming)
uses terms such as "object", "instance", "attributes" and "methods" to describe object orientation.
Such terms feel very familiar to Java and Ruby programmers. However, to truly
understand what OO is, we'll go the source, the man that coined the term, [Alan
Kay](http://en.wikipedia.org/wiki/Alan_Kay), one of the inventors of the
[Smalltalk](http://en.wikipedia.org/wiki/Smalltalk) language.

[According to Kay](http://userpage.fu-berlin.de/~ram/pub/pub_jf47ht81Ht/doc_kay_oop_en),
OO is only about message passing and isolation and protection of state.

Concepts such as classes and objects are only an attempt to implement the
ideas above. The over emphasis on classes and objects at the expense of message
passing has led many "OO" languages to stray from the core concepts of object
orientation stated above.

As Kay puts it, the spirit of OO dies when people stop thinking about objects and
the messages they send between each other and start thinking merely about "remote
procedure calls".

## Where does Erlang fit in?

If we take Alan Kay's definition of object orientation as canonical, then Erlang
fits almost perfectly. Instead of "objects" per say, Erlang uses "processes" that
are fully isolated constructs that can only communicate with each through message passing.

Unlike objects in Java, Ruby or other so-called "object oriented" languages,
Erlang processes are fully isolated from one another, so much so that catastrophic failure
in one process does not spell disaster for all the others.

Since these processes are fully isolated, they cannot share state and are
forced to explicitly send messages between themselves to get things done.

Erlang is perhaps the only language that truly embraces the ideas that Kay
puts forth as essential to object orientation: message passing and isolation.

## Conclusion

Whether you think the argument above is convincing or a big pile of crap, I
hope you take away the idea that perhaps what we all think of as object
orientation perhaps isn't OO at all at least not how it was originally
conceived.

The next time you're writing Java, Ruby or whatever OO language you use,
stop to think about how you can mold your code to be more like what Alan Kay
originally envisioned.

To see how this can have a real, practical impact on the code you write, I
recommend Sandi Metz's book [Practical Object-Oriented Design in
Ruby](http://www.amazon.com/Practical-Object-Oriented-Design-Ruby-Addison-Wesley/dp/0321721330)
which is amazing even if you've never written a line of Ruby in your life.
