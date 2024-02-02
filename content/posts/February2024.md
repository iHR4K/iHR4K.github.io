---
title: "The first month of my new project"
date: 2024-02-02T18:25:11-05:00
draft: true
---

So to be honest that's 3 weeks :) 

That went well, I have got some basic stuff on fake data: 

I added Assimp for model loading, no materials for now that's for the next month.

Buffers creation is implemented in naive way and basic rendering w/o materials but with a special indirect buffer for them, so I have only color there now.

**Node and transform systems**: I added some prepared buffers with hardcoded count, right now 128, and I can add more later if I would need, anyway I really loved idea of handles, here is the

[source]: https://floooh.github.io/2018/06/17/handles-vs-pointers.html	"source"

I was thinking about something robust, OOP and at some point I realized how I mix data and operations, and how ugly it became, so Rendering Cookbook helped me to solve this (this book really a good one, check it if you have not already :) )

After that I added some nodes which rotates in orbits relatively to each other to see how local/global space works, everything as expeceted.

**Camera interactions**: that was a little tricky, I found a good receipt in Sergey Kosarevskiy's book about camera interactions, while I love DI as a thing, that seems a little odd in pair with Handles, I prefer to use hardware manipulations directly on top of transform, anyway dumping and main code was inspired by his receipt, thank you

![](/img/img_book.jpg)

**UI layer**: There was nothing smart, I just implemented basic MVC to show scene tree and used imgui to show it

**Import/Export**: that was really interesting, while code was not hard, different types on read/write a few times took some time to debug, I never worked on scene saving before, so I really enjoyed this. In next iteration, I have to add texture arrays and dds encoding, so that will be interesting.

![](/img/jan2024.gif)

In the next month, I want to do some refactoring and start the material system, I already know which BRDF I want to try.

So on this note my first month has come to an end, really interesting how I have got that joy back, when you can't go to sleep when something doesn't look right, only can imagine what is awaiting me in the future