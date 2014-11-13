---
layout: post
title:  "Unpacking optionals with switch in Swift"
date:   2014-11-13 21:00:00
categories: blog
---
While I was working on a piece of code written in Swift I struggled with making
complicated logic with optionals in a nice way. I eventually figured how to do
it with the `switch` statement, complete with unpacking. I'll first describe
the background of the solution (in ways of examples), then the solution I came
up with. 

If you just want the **TL;DR** on how to unpack optionals with `switch`, feel
free to [skip ahead](#the-real-solution).

## The optionals problem
 
If you're working with multiple optionals (which, if you're interfacing with
Objective C, you'll do a lot of) and you're only using the `if let` syntax to
deal with it, it can get real nasty and you'll end up with code like this (or
worse):

```Swift
if let x = optx {
  if let y = opty {
    if let z = optz {
      // etc...
    }
  }
}
```

This becomes even more problematic if your logic is more straight forward than
"check if everything is set", and could lead to a fair bit of duplication. The
Swift answer to situations like these (even without optionals) is the `switch`
statement, which is more of a supercharged matching operator than the `switch`
of C-likes.

However, when optionals are involved it isn't straight forward how exactly to
take advantage of the matching in a sensible matter while not having to deal
with unpacking.

To illustrate the problem and a few non-working solutions I'm going to go through an example (similar in spirit to what I was doing), and the solutions you might try. Let's say you want to write this function:

```Swift
func checkLegality(name:String?, age:Int?) -> String {
  // to come
}
```

Inside this function you want to check if a person is legal to enter a bar (in
Norway). If they aren't (or age is unset), return `"You're not allowed!"`,
otherwise, return `"Welcome to the bar!"` or `"Welcome to the bar, name!"`
depending on whether name is set or not.

## A couple "obvious", but non-working, solutions

First, for completeness sake, this is one solution with `if let`:

```Swift
func checkLegality(name:String?, age:Int?) -> String {
  if let age = age {
    if age >= 18 {
      var nameString = ""
      if let name = name {
        nameString = ", \(name)"    
      } 
      return "Welcome to the bar\(nameString)!"
    }
  } 

  return "You're not allowed!"
}
```

It's not **that** bad, but definitely not pretty. We really want to do this
kind of thing with a `switch` (especially as the amount of parameters expand).
The first try might look something like this:

```Swift
func checkLegality(name:String?, age:Int?) -> String {
  switch (name, age) {
  case let (n, a) where a >= 18:
    return "Welcome to the bar, " + n + "!"
  case let (nil, a) where a >= 18:
    return "Welcome to the bar!"
  default:
    return "You're not allowed!"
  }
}
```

Unfortunately, this (slightly contrived to force out the bug) example gives a
compile error saying that `n` needs to be unpacked. So, even though you're
using `let`, it does not give you an already unpacked value like the `if let`
construct does. This also means that `n` might be nil at this point, so if you
had written `"Welcome to the bar, \(n)!"` for the first string, it would have
compiled, but given `"Welcome to the bar, Optional(nil)!"` when `name` is
unset. To compound the confusion, you do NOT need to worry about unwrapping
optionals in the `where` test. (This behaviour contradicts some articles
written during the XCode 6 beta which indicate it did use to implicitly nil-
check in `case let` statements)

OK, you think to yourself and head back to the Swift book, and stumble upon the
section that shows you how to do type checking in a `switch`. You then try with
this, using your newfound knowledge:

```Swift
func checkLegality(name:String?, age:Int?) -> String {
  switch (name, age) {
  case let (n, a) as (String, Int) where a >= 18:
    return "Welcome to the bar, \(n)!"
  case let (nil, a) where a >= 18:
    return "Welcome to the bar!"
  default:
    return "You're not allowed!"
  }
}
```

This compiles and seems like it should work, but quite surprisingly returns
`"You're not allowed!"` when called with `checkLegality("Name", 24)`. I can't
quite understand why that is, it seems like a bug, but it is currently the case
with XCode 6.1. Luckily, in my case, I had tests to catch this. :)

The related solution of doing `case let (n as String, a as Int)` will give you
the, seemingly incorrect, error message "is test is always true" implying that the downcast (unwrap) is always possible.

## The real solution

If none of those solutions work, is it at all possible to use `switch`
to both do logic *and* unwrap optionals in one fell swoop? The answer is
(thankfully, or this article would be a lot more depressing) "Yes!", but the
Swift book doesn't give the answer in any obvious way.

The crucial hint is that the `Optional` type acts like it is, under the hood, an `enum` like this:

```Swift
enum Optional<T> {
 case Some(x:T) 
 case None
}
```

Armed with this secret knowledge (which is implicitly hinted at in the
*Generics* chapter of the Swift book), you can write the proper solution:

```Swift
func checkLegality(name:String?, age:Int?) -> String {
  switch (name, age) {
  case let (.Some(n), a) where a >= 18:
    return "Welcome to the bar, \(n)!"
  case let (nil, a) where a >= 18:
    return "Welcome to the bar!"
  default:
    return "You're not allowed!"
  }
}
```

This works as expected, and gives you `n` and `a` as unwrapped values inside
the scope of the `case` statement. The `switch` statement doesn't seem to care
whether you use `nil` or `.None`, but the `.Some(x)` is required. Also, as a
sidenote: as I am only checking the value of `a` with the `where` statement and
not using it in the `case` block, I don't need to wrap it in `.Some()`, since a
nil value implicitly cannot pass the test.

Hopefully that helps :)
