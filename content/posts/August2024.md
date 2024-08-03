---
title: "Irradiance Volumes"
date: 2024-08-03T10:25:35-04:00
draft: false
---

I want to reset dates for my future posts, so here are the 2 weeks of my work.
I began the Irradiance Volumes implementation and almost immediately I got the issue which took ~3 weeks to understand.
![](/img/failed.png)
When you don't know exactly where the issue is you have to find a way to test everything on the GPU side, starting with input data like normals and finishing with checking every aspect of weight in calculations. Some staff were really easy to prove right, but the interpolation was not so clear. If you have 8 points, checking they are valid is not simple. I found the way how to test it, by adding a threshold, so I have one most important value. And dear PIX showed me I had different values for neighbor pixels, that was a moment I will never forget :) All the time I knew something on the GPU side was wrong, but I never blame side things until I'm 100% sure. So the main problem is this - GPU expects you to set the index as uniform for a group of pixels, and if that's not true, you have to add NonUniformResourceIndex(index).
Since I used CubeTextures array, I had an index in a group different with respect to how you see the scene, but if you use something like color from buffer - there is no problem.
For more technical terms, please read this: https://www.asawicki.info/news_1608_direct3d_12_-_watch_out_for_non-uniform_resource_index

So at the end here are the results, with data from cube textures, everything as expected.
![](/img/valid_iv.gif)

After all, I'm really happy I was able to finish this, and I can go to more advanced algorithm from Morgan McGuire.