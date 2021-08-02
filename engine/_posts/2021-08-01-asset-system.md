---
layout: post
title: \[Engine\] Adding an asset system for 3D models
---

I want to add an asset system to engine to make it easier to load 3D models into it. Currently most stuff is hardcoded which
makes it annoying to work with and add new assets for 3D models.

My first idea is to bundle everything that makes up a single model into one directory structure in the assets folder. This directory can then be considered a complete look at the pieces of that model. The basic structure is as follows: (the example model is one for a wooden bridge)

![directory_structure](https://user-images.githubusercontent.com/37605997/127804026-538cf381-64ec-4ba3-b6f7-ed639f9c0266.png)

The pros of this system would be that it is simple, easy to extend, and easy to explain. It groups assets by where they will be used so when it comes time to load the bridge model only one asset directory is accessed.

The downside to the system is that there would be asset duplication. If there are models: wooden bridge, wooden fence, and log cabin. You can imagine that all of these may make use of the same wood texture and material files, but that texture would be duplicated among the sub directories of these assets. You can also imagine this happening for meshes. Say if you had models for wooden boxes, stone blocks, and metal boxes. These all might just have a cuboid mesh for the model. but that mesh will be duplicated among all three of the models' directories.

The progression then from the previous idea is to create some meta data that does the work of the directory structure and
then we can group up all the assets of similar types and access them by tags in the meta data. My idea is that a JSON
file would replace the idea of the model_name directory and inside it would be a number of fields keyed by what type of asset
they are and containing the file name of the assets they need. For example the wooden bridge in the new system might look
like this:

```json
{
    "meshes": ["wooden_plank.obj", "log.obj", "rope.obj"]
    "textures": ["wood.png", "rope.png"]
    "normalmap": ["wooden_plank_norm.png", "log_norm.png", "rope_norm.png"]
    "materials": ["wood.mtl", "rope.mtl"]
}
```

This solves the duplication problem while still being fairly simple I think.