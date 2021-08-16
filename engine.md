---
layout: page
title: engine
---

Engine is a project for me to learn about how games are made while improving my C/C++ knowledge [GitHub](github.com/Andidy/engine)

I originally based engine on the early episodes of [Handmade Hero's](https://handmadehero.org/) first 20-ish episodes but once I
had the basic understanding of how to get something on screen and some input from keyboard and mouse I branched out to exploring
different graphics APIs. I used OpenGL, which I had used at UMN for graphics courses and with the 
[Raylib framework](https://www.raylib.com/) for hobby projects. Eventually, I wanted to explore other options. I spent a bit of
time with Vulkan doing the "hello triangle" but I found it was too much API to get things done, this is when I started working on
the [carnegie project](https://andidy.github.io/carnegie) and found similar issues to Vulkan with the API holding me back from
learning the basics of graphics programming. I settled on DirectX 11 as it is built for the OS I develop on and some veteran
programmers I've spoken with have told me that it has less issues with drivers and shipping to other people.

Once I settled on DirectX 11 I've started building engine out so that now, though its more of a framework than an engine,
it includes a decent little feature set:
- (Partial) Mouse and Keyboard Input
- (Partial) .OBJ Model loading
- Texture Loading through stb_image
- 3D rendering
  - (Hardcoded) Blinn-Phong Shading
  - Lighting done in pixel shader
  - Single texture per model
  - Rendering data is seperated from gameplay data
- Object Picking for rendered entities (very basic hitbox right now)

![Textured Models with Basic Lighting](/docs/assets/images/engine/textured_models_basic_lighting.png)

[![Object Picking](/docs/assets/images/engine/yt_object_picking.png)](https://www.youtube.com/watch?v=XoNrCGmb1VI)

[Link to the bunny model and texture used for the preview image](https://opengameart.org/content/hand-painted-bunny-unrigged-version)

For the deer model and tree model I don't remember where I got them from but I remember they were open/permissive licenses.

[Back to homepage](https://andidy.github.io/)