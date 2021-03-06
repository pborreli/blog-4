---
title: "What's All This Fuss About Monads? λ"
date: 2014-02-20T19:30:00+01:00
---

Despite Functional Programming's (FP) growing popularity, for many "normal
programmers" the language family seems to be merely an academic exercise that
they are neither capable of or willing to engage in. Despite there being many FP
languages that are arguably much more accessible than some "normal" programming
languages like C++ (e.g.  Elixir), FP does at times have some concepts that remain
fuzzy even to those who have programmed before in the functional style.

Perhaps no concept breads more fear in the hearts of aspiring functional programmers
than that of the monad. In this series of posts, I'll try my best to explain what a
monads are and more importantly, why they're awesome.

This won't be a formal attempt at defining monads but rather a practical
look at the expressive power they give the programmer using them. In other
words, if you're looking for some category theory, look elsewhere.

## No PhD Required

In order to get the most out of this article you should be somewhat comfortable with the
basic tenants of functional programming (especially first class functions), and you
should feel comfortable with static typing. Don't worry though. Even if you've never
touched a FP before, you should still be able to get something out of this too.

## Our Journey

On our road to understand what the heck a monad is we'll also be getting to
know other concepts with seemingly scary names like functors, applicative functors, and
monoids.

Accompanying us on our journey will be the Haskell programming language. Don't worry
if you've never programmed in Haskell, I'll try to explain everything you need to
know along the way. Also, this will not be an academic explanation of what a monad is.
If you're looking for your answers based in Category Theory, you'll have to search
elsewhere.

## Yo Dawg. I Heard You Like Values...

Before we get started, it's best to look ahead to where we are going. Haskell
is very statically and strongly typed. `"hello"` is a `String` and `1.0` is a
`Double`. When we declare functions we will specify exactly the kinds of types
that we will alow to be passed into and returned from our functions. The compiler
will make sure we keep this in mind the entire time we program.

Haskell's type system is in fact more pervasive than perhaps you're
used to in other languages like Java. Even actions which have side effects (e.g.
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
types like `String`s, `Char`s, and `Integer`s. In Haskell you can also have
types that are part of the `Monad` type class. Monads let you have values that
are wrapped in a context. This context tells you more about the plain value
inside.

Let's take a look at an example to clarify.

Haskell has a type called `Maybe` which is part of the type class `Monad`.
`Maybe` wraps another type (like a `String` or `Integer`) and gives it a context
(or more information) about that plain type inside. What context does `Maybe` give?

Maybe tells us that a value is either there or it is not. In other words,
we maybe have a value.

Here are three instances of the Maybe type:

```
Just 5
Just ["Hello", "World"]
Nothing
```

The first two values tell us we do indeed have values, the `Integer` 5 in the
first case and the list of `String`s "Hello" and "World" in the second case. The
third value says we have nothing at all.

Scala (another FP style language) has a similar type called option that looks
like this:

```
Some 5
Some ["Hello", "World"]
None
```

We either have some of something or none. The `Maybe`/`Option` monad gives us
the ability to say we have some sort of value or we do not.

## Get Func-y

Of course, these values in isolation don't do us much good. Of course, if we
manually write out the value `Just 5`, we know that we have the `Integer` 5.
Things become much more interesting when we have functions.

So let's say we have a function (which we'll call `myFunction`) with the following type signature:

```
myFunction :: Integer -> Maybe Integer
```

This says that the function `myFunction` takes in an `Integer` as its only
argument and returns a `Maybe Integer`.

So we, and the compiler, know that when we pass in an `Integer` to myFunction we
"maybe" will get an `Integer` back. Depending on our input we can get back out
something like Just 5, or Just 49283 or even Nothing (which, of course, is different than
Just 0).

The Maybe monad is often used where in other languages we would use the value
nil (or null). However, for reasons that we will explore later, the Maybe monad
allows us to do the same sort of things we would do with nils without having to
always do nasty `if nil` checks. The compiler guarantees that you'll never have
any `undefined method 'foo' for nil` type errors again.

Not only can we avoid nil checking, but the compiler will catch any mistakes we
make at compile time.

## Other Monads

So we now have a monad that gives a wrapped value with the context of either being
there or not. `Just "Hello"` or `Nothing`. What other "contexts" can we wrap values
in?

* IO - IO wraps a value in a context where side effects (the bane of
  functional programming) can take place.
* Writer - Writer Wraps a value in the context of having another value that
  acts as a "log". This is often used to log the intermediate steps of a
  function.
* List - Yes, `List`, the data structure backbone of almost all functional
  programming, is a monad with the context of having many values or having none
  at all.
* And many more!

## What's Next

Now that we know where we're headed, we'll start next time looking at two
concepts central to monads, functors and applicative functors. All monads are
both functors and applicative functors so understanding both will get us much
closer to understanding monads.
