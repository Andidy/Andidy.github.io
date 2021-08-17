---
layout: post
title: Engine Further discussion of the Asset System
---

Before discussing the asset system and how to make it nice to use, I thought I would give an overview of the current way assets
flow through the program and how the game data gets to the renderer to be drawn.

***

Right now I use a system where my "game things" store their rendering positions, rotations, and scales that the game loop
will update and then after game update happens these get pulled out into a renderer compatible format that adds in the data
of textures and meshes, and then the renderer loops over this array of renderer friendly structs and uses them to draw the scene.
I have in my GameState struct an array of "entities":
```cpp
struct Entity {
    vec3 render_pos;
    vec3 render_scale;
    vec3 render_rot_axis;
    f32 render_rot_angle;
};

struct GameState {
    i32 num_entities;
    int blackGuyHead;
    int blackGuyHead2;

    int bunnyTest;
    int bunnyTest2;

    int cubes[7];
    int quad;

    int MAX_ENTITIES;
    Entity* entities;
    
    // camera, object picking state, allocators etc.
};
```
And then I initialize my "entities" before entering my main loop something like this:
```cpp
gs->entities[gs->blackGuyHead = gs->num_entities++] = { Vec3(-1.0f, 0.0f, -1.0f), Vec3(1.0f, 1.0f, 1.0f), UpVec(), 0.0f };
```
And then inside the game update function this "entity" could be updated with keyboard input or particle system or AI or whatever.
Then before the rendering occurs I "PrepareRenderData" by copying parts of the "entities" into a new struct that also contains
mesh and texture information, notice the inclusion of some hardcoded numbers at the end:
```cpp
// types
struct RenderEntity {
    vec3 pos;
    vec3 scale;
    vec3 rot_axis;
    f32 rot_angle;

    int32_t model_index;
    int32_t texture_index;
};

struct RenderData {
    int32_t num_entities;
    RenderEntity* entities;
};    

// usage
e = gs->entities[gs->bunnyTest];
render_data->entities[iter++] = { e.render_pos, e.render_scale, e.render_rot_axis, e.render_rot_angle, 2, 2 };
```
Then in my "RenderFrame" function I use the RenderData to draw the "entities":
```cpp
for (int i = 0; i < render_data->num_entities; i++) {
    RenderEntity re = render_data->entities[i];
            
    context->PSSetShaderResources(0, 1, &texture_views[re.texture_index]);
            
    mat4 translate, scale, rotate;
    translate = TranslateMat(re.pos);
    scale = ScaleMat(re.scale);
    rotate = RotateMat(re.rot_angle, re.rot_axis);

    mat4 world = MulMat(translate, MulMat(rotate, scale));

    // sending matrices to shader in constant buffer

    context->VSSetConstantBuffers(0, 1, &constant_buffer);
    context->DrawIndexed(m_buffer->models[re.model_index].length, m_buffer->models[re.model_index].start_index, 0);
}
```
Here you see I call DrawIndexed on m_buffer, m_buffer is what I call a ModelBuffer and its an array of slices into my index buffer.
I do this because I pack all my vertices and indices into one big buffer during initialization since it is more clear and less
overhead than having to rebind different vertex and index buffers. Here is the structs for the Vertex, Index, and Model Buffers:
```cpp
struct VertexBuffer {
    i32 buffer_length;
    i32 num_vertices;
    Vertex* vertices;
};

struct IndexBuffer {
    i32 buffer_length;
    i32 num_indices;
    i32* indices;
};

struct Model {
    i32 start_index;
    i32 length;
};

struct ModelBuffer {
    i32 buffer_length;
    i32 num_models;
    Model* models;
};
```
Currently the way these buffers are initialized and loaded with assets looks like this:
```cpp
InitGameState(&game_memory, Vec2((f32)dim.width, (f32)dim.height));

VertexBuffer vertex_buffer;
IndexBuffer index_buffer;
ModelBuffer model_buffer;
PermanentResourceAllocator renderer_allocator(Gigabytes((u64)4));
FrameAllocator frame_allocator(Megabytes((u64)64));

InitRenderer(&vertex_buffer, &index_buffer, &model_buffer, &renderer_allocator);

// meshes are loaded
LoadOBJ((char*)"test_assets/african_head.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
LoadOBJ((char*)"test_assets/deer.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
LoadOBJ((char*)"test_assets/bunny.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
LoadOBJ((char*)"test_assets/tree_default.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
LoadOBJ((char*)"test_assets/cube.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
LoadOBJ((char*)"test_assets/quad.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);

// the font for text rendering is loaded

int n, iter = 0;
Image* images = (Image*)renderer_allocator.Allocate(sizeof(Image) * 16);

// textures are loaded
images[iter].data = stbi_load((char*)"test_assets/african_head_diffuse.tga", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/grass.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/bunny.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/red.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/orange.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/yellow.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/green.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/cyan.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/blue.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/gray.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = stbi_load((char*)"test_assets/no_tex.png", &images[iter].width, &images[iter].height, &n, 4);
iter += 1;
images[iter].data = font_image.data;
images[iter].width = font_image.width;
images[iter].height = font_image.height;
iter += 1;

// text rendering stuff is initialized

Renderer renderer = {};
renderer.InitD3D11(window, dim.width, dim.height, &vertex_buffer, &index_buffer, images, iter, text_verts, NUM_TEXT_VERTS);

for (int i = 0; i < iter; i++) {
  stbi_image_free(images[iter].data);
}

RenderData render_data = {};
render_data.entities = (RenderEntity*)renderer_allocator.Allocate(sizeof(RenderEntity) * (i64)(((GameState*)game_memory.data)->num_entities));
```
Remember the hardcoded numbers in the RenderData structs earlier?
```cpp
render_data->entities[iter++] = { e.render_pos, e.render_scale, e.render_rot_axis, e.render_rot_angle, 2, 2 };
```
Those numbers are based on the order that LoadOBJ and stbi_load are called in the preceding code.

***

Now that we've discussed the current state of the engine, I will explain what I think is a good path going forward.
I asked around on discord how other people handle code like this. They suggested I store a handle in the game "Entity" structure
which indexes into my storage of my meshes, textures and other rendering assets. My understanding of this is as follows:
```cpp
enum AssetType {
  UNKNOWN,
  MODEL,
  MESH,
  TEXTURE
};

struct AssetHandle {
  char* name;
  AssetType type;
  int handle;
};
```
The name is a string used to identify the handle in a human-friendly way (Ex. "bunny", "cube", "grass"), the handle is the index
into the asset array containing the specific asset, and the type is to help in cases where there would be name collisions.
These asset handles could be stored in an array and during the initialization of the GameState the handles could be passed to
the game state and then stored as one of the Entity fields. When the game then passes the entities to the function that converts
them to the RenderData format, these asset handles can be used to look up what model and textures that entity needs.
```cpp
// new entity
struct Entity {
    vec3 render_pos;
    vec3 render_scale;
    vec3 render_rot_axis;
    f32 render_rot_angle;
    AssetHandle model_handle;
};

// new conversion
render_data->entities[iter++] = { e.render_pos, e.render_scale, e.render_rot_axis, e.render_rot_angle, e.model.handle };
```
I imagine the population of the asset handles would look something like this in the current setup:
```cpp
AssetHandle assetHandles[];

i = 0, mesh_index = 0, texture_index = 0;

assetHandles[i++] = { "african_head", mesh_index++ };
LoadOBJ((char*)"test_assets/african_head.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
// more meshes

assetHandles[i++] = { "african_head", texture_index };
images[texture_index].data = stbi_load((char*)"test_assets/african_head_diffuse.tga", &images[texture_index].width, &images[texture_index].height, &n, 4);
texture_index++;
// more textures
```
And to make it more dynamic eventually something like this:
```cpp
entity_data = LoadEntityData("path/to/entity/data");

AssetHandle assetHandles[];

for (entity in entity_data) {
  if (not_loaded(entity.model.name) {
    assetHandles[current] = LoadModel(entity.name);
  }
  if (not_loaded(entity.mesh.name)) {
    int index = LoadOBJ((char*)"test_assets/african_head.obj", &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
    assetHandles[current] = {entity.mesh.name, MESH, index};
  }
  if (not_loaded(entity.texture.name)) {
    images[texture_index].data = stbi_load((char*)"test_assets/african_head_diffuse.tga", &images[texture_index].width, &images[texture_index].height, &n, 4);
    assetHandles[current] = {entity.texture.name, TEXTURE, texture_index};
  }
  
  game_entities[current] = { entity.pos, entity.rotation, entity.scale, assetHandles[current] };
}
```

***

I think this is a good enough jumping off point and tomorrow after work I will be attempting to implement and test it out.