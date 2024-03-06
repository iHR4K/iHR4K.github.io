---
title: "Keep Normals Normal"
date: 2024-03-05T18:25:11-05:00
draft: true
---

![](/img/screenshot.jpg)

Small reflection on this month, not so bad, I spent some time starting implementing shape lists, textures reading, and converting to DDS, I was able to load Sponza and curtains from the additional scene, 
later I added the opportunity to move elements and scenes and save the scene and all caches I have for materials, shapes, etc, reading my own format much faster, so that's the experience I was looking for.
Some small interesting things I found on the way, for example really interesting how FBX stores transparent materials - there is no way to get it from Assimp, only if you support their material shader, so if you are in this problem - just use glTF, if you need FBX - be ready to support full shader, I would like to do this at some point, but not now,
I have a good knowledge of BRDFs already, I chose the Disney BRDF term and took GGX for rest, that's a good start so I was able to see some lights in dynamic way, at some point to try that fresh Fresnel term from Unit, looks promising https://belcour.github.io/blog/research/publication/2020/08/26/brdf-fresnel-decompo.html
Immediately I realized how bad 4k textures in long distances and I have to do something with it - I have looked into LEAN mapping, but decided to move forward for now because my main target for now is the Global Illumination Light maps or something like this, 
I know already which materials I want to see in my future game, something similar to Mirro's Edge, good DICE showed a lot of info about this game, so I will be researching this next month,
For texture encoding, compression I used https://github.com/microsoft/DirectXTex - an amazing framework, 
For nodes transform manipulation - https://github.com/CedricGuillemet/ImGuizmo
Thank you, guys, for keeping this OpenSource, you are great :)


End of the month: I started working on reflections probes, and how I can cache them, so I can use those methods in the future for all probes - Distant Light, Local Light, Planar and Screen Space Reflection Probes
Sp next month should be really interesting

Probably I will need to check the integration of Intel Embree, we will see

February was a rough month, RIP Navalny