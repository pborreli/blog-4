---
layout: post
title: "Babel"
date: 2014-05-26 22:07
comments: true
categories:
---

*And the Lord said, Behold, the people is one, and they have all one language*

Linguistic diversity is one of the true treasures of the human species.
Learning a language
imparts to us new insights, new ways of viewing the world.

Programming languages often times mirror their natural language counterparts
when it comes to what can be gained from learning new ones. Learning new
languages
can be extremely informative, giving the programmer not only new paradigms to
think
in but also a new vocabulary which she can use to describe concepts present in
languages already learned.

At 6Wunderkinder we employ an ever growing range of programming language tools
to get our job done.
With the risk of forgetting a language, I believe we currently employ 11
languages in total. They
are:

* Ruby
* Java
* Objective C
* C#
* Javascript
* Scala
* Clojure
* Haskell
* Python
* Go
* Bash

## Reasons

The reasoning for why we chose each of these languages varies wildly.

### Necessity
Some languages we were forced to choose. Java, Objective-C and Javascript all
dominate their
respective platforms. While we could have, for instance, chosen Scala for
Android Development or Dart for the Web, these choices just didn't make sense.
The internal developer knowledge coupled with the community support for the
dominant languages
made the choice rather easy.

### Development Speed
Some languages were chosen for their development speed. While building new
features, we often went for languages that we knew best and that naturally lend
themselves to faster development. Ruby is probably the best example of such a
language. We all have experience writing Ruby, and the language itself is
perfect
for getting something running fast. When we need a first implementation,
  a prototype or even pseudo-code, we turn to Ruby.

### Performance
Some languages were chosen for their performance. Our team is still quite new
to the
worlds of Scala, Clojure and Go, but these languages called to us because of
what they could afford us in raw speed. As amazing and fun as Ruby is
to write at the end of the day, it costs more to run Ruby not only in terms of
performance but also in terms of number of servers needed. The aforementioned
languages allow us to write performant applications and maintain (or even gain)
expressiveness.
Additionally, in some cases (especially Scala and Clojure), the languages
themselves
can spread new light on the art of programming itself.

### Edification
Some languages were chosen for learning new ways of developing software. I've
been
dabbling in Haskell for quite some time now. One day we needed a small but
highly performant web service. I decided to write the app in Haskell.
The app gave me and a co-worker insights into a completely new way to make
software.
This knowledge has made us a lot more productive when working
with other languages. Also it certainly didn't hurt that the Haskell
implementation
could handle over 7 times more requests per second than an implementation
written in Ruby.

## Challenges

Having new languages deployed into production is not without its issues. I'll
use the Haskell app as an example. I want to see Haskell succeed at
6Wunderkinder, because I find it a joy to work on. I'm fully aware that besides
myself and my co-worker, there is no one who is currently capable of
maintaining or
improving the app. Haskell has a big learning curve and everyone on our team is
busy
enough without having to take time out of their schedules to learn Haskell. But
there are a few reasons why I don't regret writing my app in Haskell:

### More Opportunities to Learn
The Haskell app gives my fellow developers an excuse to get into Haskell if
they want to.
Just as much as I'm now afforded the opportunity to explore Scala and Clojure
at work,
my colleagues can now do the same with Haskell.

### Easy to Rewrite
The web front end for the app is a mere 40 lines of code. If there's really a
need
to add a feature and my co-worker and I are not there to help, it's fairly
trivial to
just rewrite the service. In fact, to benchmark the app, we wrote two other
versions
of it in Ruby and Go.

### Good for Developer Culture
I want to work in a company where the freedom to choose the right tool for the
job
is cherished. I want to be trusted to make the right decision for my team and
for my company. In fact I've not written apps in other languages like Elixir
  and
  Rust because I'm not quite ready to justify the investment to my team.
  If and when the day comes when I feel comfortable doing that I'm glad I'll be
  able to.
  In the end the choice is ours.

### An Objectively Good Choice
I didn't choose Haskell just because I thought it would be fun. Haskell's type
safety
and speed made it a smart fit. Given the choice, competent developers will pick
the
right tools for the job.

The dive in polyglotism is relatively new at 6Wunderkinder. There lies a lot of
challenges ahead about how we ensure that we don't let our freedom bite
us in the butt. We're still not completely sure that it's the right choice for
us. However, given the benefits that have come from the freedom so far,
I'm confident that things will turn out alright.
