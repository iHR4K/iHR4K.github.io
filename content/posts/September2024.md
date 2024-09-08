---
title: "Irradiance Fields"
date: 2024-09-08T11:03:13-04:00
draft: false
---

![](/img/irrad_fields.gif)

Irradiance Fileds is a pretty fresh approach to the real-time GI, I wanted to implement it for quite a long time and finally, I came to that point.
So while the main algorithm is really simple, implementation can bit tricky at some points, also if you never worked with a few things like octohedral encode/decode or DirectX Ray Tracing. You have 2 options to start - one with a simple implementation of the update routine, when you compute everything for every probe, or you can go with another implementation from a late Majercik article (also don't be like me - always check publications supplement materials, I have not checked and spent a lot of time on some utils implementation, but Majercik provide almost everything in zip archive https://jcgt.org/published/0008/02/01/supplement.zip)
I implemented the first version and almost immediately got an issue with weight calculation, so after basic proof, I switched to Majercik's approach with full trace-inline implementation in compute shader.
Here is one tip, use the same target formats as publication, I used RGBA8 for one because that was easy for me to debug it, but you can't interpolate to zero with that(not sure that's common fact), by the way GRBAFloat16 is available in swapchain configs.
Also at the beginning I wanted to find something to use as a base and found this one repo - https://github.com/DaOnlyOwner/hym, it shows some basic usage of the technique. It was out of date so I spent some time to recover it - https://github.com/iHR4K/hym, just in case if you want to run something.

At the same time, I wanted to check some information about DX compilator and found it's under Xbox NDA, so I registered in ID@Xbox. That went really bad by the way there were a couple of people from support team who helped me a lot. So Thank you so much, Matt Hanson.
By the way, to complete registration, you need to show the build, which I don't have now. Will continue this journey in a couple of months.

After all that's time to finish GI research for now, I will complete Majercik's article and will switch to other things. Memory management and OpenGOAL await me :)
