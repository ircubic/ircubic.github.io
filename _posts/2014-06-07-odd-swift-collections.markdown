---
layout: post
title:  "Odd Swift collections behaviour"
date:   2014-06-07 18:00:00
categories: blog
---

Apple has released its new language, Swift, on the unsuspecting beta masses, and
it's pretty swell. However, there's an oddity in the behaviour of its collections
(dictionaries and arrays) that deserves to have some light shone on it.

The details are hidden away in an obscure corner of the Swift book, with the
surprisingly boring and inconspicious title "Assignment and Copy Behavior for
Collection Types". The gist of it is to define what happens when you use a
`let` vs a `var` for a Dictionary or an Array, as well as what happens when
passing Collection types to functions.

Dictionaries work like expected, put them in a `let` and you can't change them
in any way, but an Array is another story entirely. If you put an Array in a
`let` you *can* change it, but *only* with operations that don't alter its
size! This means you can do this:

```
let array = [1, 2, 3, 4]
array[2] = 200 // array is now [1, 2, 200, 4]
```

Arrays get even weirder when you send them as a parameter. Dictionaries get
automatically copied so you can't stomp over the contents of the source array
accidentally. Arrays, however, retain the weirdness from before, and only get
copied when you execute an operation that alter its length. This means that
this function will probably not do what you expect:

```
func doThing(var array:Array<Int>) -> Array<Int> {
	array[0] = array[0]*2
	array.append(array[1]*2)
	return array
}

let originalArray = [1, 2, 4]
let otherArray = doThing(originalArray)
```

If you now try to print out the contents of the arrays, you get this:

```
println(originalArray) // [2, 2, 4]
println(otherArray)    // [2, 2, 4, 4]
```

As you can tell, not only did the array you sent in got changed, but the array
returned is different from the source array. This is because Array instances
automatically make an internal copy when something is done to them that
changes their length (`append()`, `insert(,atIndex:)`, `removeAtIndex()`,
among others).

The solution to this problem is to call the `unshare` method on the array,
which will create a copy if the array has more than one reference. You could
also call `copy` for much the same effect, but it will create a copy regardless
of whether one is needed or not. Doing this gives you the following:

```
func doThing(var array:Array<Int>) -> Array<Int> {
    array.unshare()
	array[0] = array[0]*2
	array.append(array[1]*2)
	return array
}

let originalArray = [1, 2, 4]
let otherArray = doThing(originalArray)
println(originalArray) // [1, 2, 4]
println(otherArray)    // [2, 2, 4, 4]
```

It's one of those things that, once you know it, it's easy to deal with, but it
is by no means obvious and could potentially lead to hard to debug bugs. So
there you go.
