---
title: Making an overclock knob with nScope
layout: post
date: 2016-04-26 19:42
categories: blog
---

I finally received my [nScope](http://nscope.org) lab kit from backing the
project on kickstarter, and started playing around with it and its API. Even
despite its lack of documentation, it's surprisingly straight forward, giving
you access to four signal generators (two analog and two pulse) which can also
be used as outputs, four analog inputs, and the ability to easily power a
breadboard via USB.

After toying with some of the components they shipped with it and making
some blinkenlights and various simple circuits, I realized that they shipped
me a potentiometer (a "volume knob"-type component). Now, I've spent the last
few weeks reverse engineering and developing a library for accessing GPU
overclock functions ([lib_gpu](https://github.com/ircubic/lib_gpu/)), and this
seemed like a perfect opportunity to put it to an actual physical use.

So I trivially wired up the potentiometer to feed its output straight into
channel 1 of the nScope, like so:

![Wiring schematic for potentiometer](/images/nscope_gpu.jpg)

Then I cloned the [nScope API](https://github.com/nLabs-nScope/nScopeAPI) and
started editing the `program.py` file found in the python folder to make a
dummy program that wires together the input from nScope and feeds it into my
`lib_gpu` library:

<script src="https://gist.github.com/ircubic/c989f564fbf1ff7b5d104a65debfc713.js"></script>

The way the code works is pretty straight forward: the potentiometer feeds a
value between 0.0 and 5.0 into nScope channel 1, which the code polls once
every so often. This value is then converted into a percentage (rounded to
nearest 5%) and multiplied with the hardcoded maximum GPU Core MHz offset
(nVIDIA overclocking is based on offset values, not absolute values) to get
the new value. This lets me select the overclock from +0 MHz to +150 MHz
(which happens to be the highest stable value for my card) using the
potentiometer as a "GPU speed knob."

The program took me about 15 minutes to cobble together (thank you, time spent
on the simple FFI interface to `lib_gpu`) and is spectacularly useless.
However, it was fun and showcases some minor usage of the nScope API and my
own `lib_gpu` library. Plus I get to turn a nifty blue knob to overclock my
GPU, how cool is that?
