---
layout: post
title: "What's All This Fuss About Monads?"
date: 2014-02-20 19:30
comments: true
categories: fp functional programming monads
---

Despite Functional Programming's (FP) growing popularity, for many "normal
programmers" the language family seems to be merely an academic excerise that
they are neither capable of or willing to engage in. Despite there being many FP
lanuages that I would consider much more accesible than a normal programming
language like C++ (e.g.  Elixir), FP does at times have some concepts that remain
fuzzy even to those who have programmed in the functional style. Perhaps no concept
breads more fear in the hearts of aspiring functional programmers than that of the Monad.
In this post, I'll try my best to explain what a Monad is and, more importantly,
why it's awesome.

## Who This is Intended For

In order to get the most out of this article you should be somewhat comfortable with the
basic tenants of functional programming (e.g. first class functions and
immutable data), and you should feel comfortable with static typing. Don't
worry though if you're a Rubyist who has never touched a FP before. Hopefully
you'll be able to get something out of this too.

## Our Journey

On our road to understand what the heck a Monad is we'll also be getting to
know what Functors and Applicative Functors are. Accompanying us on our journey
will be the Haskell programming language. Don't worry if you've never
programmed in Haskell, I'll try to explain everything you need to know along
the way. Also, this will not be an academic explanation of what a Monad is. If
you're looking for your answers based in Category Theory you'll have to search
elsewhere.

## A Look Ahead

Before we get started, it's best to look ahead to where we are going. Haskell
is very statically and strongly typed. `"hello"` is a `String` and `1.0` is a
`Double`. The compiler will make sure we keep this in mind the entire time we
program.

Haskell's type system is in fact more prevasive than perhaps you're
used to in other languages like Java. Even actions with have side effects (e.g.
printing something to the screen or making a network call) have special types.

Let's start with a simple value:

```
ghci> let myString = "Hello, World!"
"Hello, World!"
```

If we ask Haskell the type of the value myString:

```
ghci> :t myString
myString :: String
```

We see that it is indeed a `String`. By now you should be saying, "Wow this
Haskell thing is easy!". Haskell's type system let's you do more than just have
types like Strings, Chars, Integers and Functions. In Haskell you can also have
types that are part of the Monad Type Class. Monads let you have values that
are wrapped in a context. This context tells you more about the plain value
inside.

Let's take a look at an example to clarify.

Haskell has a type called Maybe. Maybe wraps another type (like String or
Integer) and gives it a context (or more information) about that plain type
inside. What context does Maybe give? Maybe tells us that a value is either
there or it is not. In other words, we maybe have a value.

Here's an instance of the Maybe type:

```
Just 5
```

This tells us that we indeed have a value. In fact, we just have 5. Maybe can
also describe the absence of a value:

```
Nothing
```

So let's say we have a function with the following type signature:

```
myFunction :: Integer -> Maybe Integer
```

This says that the function `myFunction` takes in an Intger as its only
argument and returns a Maybe Integer.

So we, and the compiler, know that when we pass in an Intger to myFunction we
"maybe" will get an Integer back. Depending on out input we can get back out
something like Just 5, or Just 49283 or even Nothing (which, of course, is different than
Just 0).

The Maybe Monad is often used where in other languages we would use the value
nil (or null). However, for reasons that we will explore later, the Maybe Monad
allows us to do the same sort of things we would do with nils without having to
always do nasty `if nil` checks. The compiler gurantees that you'll never have
any `undefined method 'foo' for nil` type errors again.

We'll be able to take the following code equivalent:

```
def myRubyMethod(foo, bar)
  if !foo.nil?
    if !bar.nil?
      foo + bar
    end
  end
end
```

And instruct the Haskell compiler to just "do the right thing".

## Other Monads

So we know have a Monad that gives a wrapped value the context of either being
there or not. Just "Hello" or Nothing. Whatever "contexts" can we wrap values
in?

* Either - Either takes two wrapped values and presents them as alternatives to
  each other
* IO - IO wraps a value in a context where nasty side effects (the bane of
  functional programming) can take place.
* Writer - Writer Wraps a value in the context of having another value that
  acts as a "log". This is often used to carry debug messages around.
