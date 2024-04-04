---
title: "Steps to probes"
date: 2024-04-03T21:29:08-04:00
draft: false
---

![](/img/pic.png)

This month was full of surprises, I never worked on big scenes so reflection probes were a little difficult at first from the how-to-work/update standpoint, but at some point that went well, 
So I was able to add runtime probes and store them in the cache, later I added them to the calculation of the irradiance map by Karis, and I fully implemented his article, really interesting how that's pretty clear hwo to work with them in the viewer, but the bigger scene is full of surprises, like parallax, interpolation, etc.
I used Computer Shaders for probes readback implementation.
In the end I used DirectXTex, to store cached texture files.
Now a new wave of refactoring awaits me because I integrated cubemaps in a hacky way.

**Mirror's Edge research**
I have started my research about Mirror's Edge, and I think this game has amazing technical and artistic sides.
Can recommend checking this talk - Fabien Christin Lighting Mirror's Edge Catalyst (https://www.gdcvault.com/play/1023284/Lighting-the-City-of-Glass). Seems like they used a pretty old but good solution - light maps for big objects and SH for small, so I will go in the same direction.

So next steps I want to refactor the Editor's code to something reusable, so I can easily add new renderers w/o working with heaps.
That will be my next step - I already know how to implement SH, but lightmaps could be tricky, maybe next time I will share some useful details.

**PS**: I got a really interesting bug when everything seemed right but only the poles of the sphere were wrong, I rechecked everything, the problem was in 2.0*PI+Xi.x. Should be 2*PI*Xi.x