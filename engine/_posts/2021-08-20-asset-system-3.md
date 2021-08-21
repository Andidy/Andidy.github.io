---
layout: post
title: Engine Asset System 3 Implementation
---

Over the past few days I've worked on implementing the ideas I talked about in 
["Further discussion of the Asset System"](https://andidy.github.io/engine/2021/08/16/asset-system-2.html).
The system I ended up with is similar to the final code snippet shown in the previous article but it seperates
the loading of the entities from the loading of the models. The primary benefit of this change has been making
the project less reliant on hardcoded numbers and strings which over time become difficult to manage and change.

I created the asset handles to give the game layer some indirection from the specific implementation of where the assets are.
```cpp
enum class AssetType {
	UNKNOWN,
	MODEL,
	MESH,
	TEXTURE,
	NUM_TYPES
};

struct AssetHandle {
	std::string name;
	AssetType type;
	int32_t handle;
};
```
I added a field to the game's entity structure for a model's asset handle and created a game function that is defined in the
platform layer to load the entity data from JSON file.
```cpp
void LoadGameAssets(GameState* gs, AssetHandle* asset_handles);
//
void LoadGameAssets(GameState* gs, AssetHandle* asset_handles) {
	debug_ReadFileResult file = debug_ReadFile((char*)"test_assets/entities.json");
	if (file.data != NULL && file.size >= 0) {
		std::string json_err_str;
		json11::Json json = json11::Json::parse((char*)file.data, json_err_str);

		int entity_index = 0;
		while (json[entity_index].is_object()) {
			json11::Json::object je = json[entity_index].object_items();

			Entity e = {};

			auto pos = je["pos"].array_items();
			e.render_pos = Vec3(pos[0].number_value(), pos[1].number_value(), pos[2].number_value());

			auto scale = je["scale"].array_items();
			e.render_scale = Vec3(scale[0].number_value(), scale[1].number_value(), scale[2].number_value());

			auto rotation_axis = je["rotation_axis"].array_items();
			e.render_rot_axis = Vec3(rotation_axis[0].number_value(), rotation_axis[1].number_value(), rotation_axis[2].number_value());

			auto rotation_angle = je["rotation_angle"].number_value();
			e.render_rot_angle = (float)rotation_angle;

			auto asset_name = je["model"].string_value();
			for (int i = 0; i < 64; i++) {
				if (asset_name.compare(asset_handles[i].name) == 0) {
					e.h_model = asset_handles[i];
				}
			}

			gs->entities[entity_index++] = e;
		}
		gs->num_entities = entity_index;
	}
}
```
The primary downside to this way of loading entity data is that it requires hardcoded strings for the different fields
of the struct and to match up with the JSON. Eventually, I would like to make a tool that can do that part for me.
You'll notice that an AssetHandle* is one of the arguments to this function, this is the array of asset handles where the loaded
textures, models, and meshes reside.

Originally, to load the assets I would put an individual bit of code specific to that asset in the initialization code of engine.
These assets weren't grouped at all and so magic numbers based on the order in which assets were loaded were used to tell the
renderer what assets were needed for each entity during preparation of the render data. This, obviously, wasn't ideal and so during the first
iteration of the asset handling code I created the asset handles at each hardcoded asset load point and then hardcoded passing the assets in the
right order. However, this could then be easily improved by loading the models.json file and using that data to group seperate assets under
one handle for the entities to hold onto. The models were being loaded after the assets however, which meant that the hardcoding was still a
requirement and that unused assets could be loaded. The final implementation fixes this by first loading models.json, and then iterating
through the models and checking if their assets have been loaded. If the asset has been loaded that handle is retrieved and given to the
model. If the asset hasn't been loaded then I load the asset and generate the asset handle to give to the model. Once all models and all assets
have been loaded, I load the entities which will use entities.json to figure out which model they should be basing their data on.

Below is the code as it stands. Still has some hardcoded numbers and strings but they don't grow without adding more asset types, where as before
hardcoding grew with each entity.
```cpp
const int MAX_ASSET_HANDLES = 64;
AssetHandle asset_handles[MAX_ASSET_HANDLES];
int asset_index = 0;

std::string asset_folder_path = "test_assets/";
std::string asset_path = "";
const int MAX_MODELS = 64;
Model models[MAX_MODELS];
debug_ReadFileResult file = debug_ReadFile((char*)"test_assets/models.json");
if (file.data != NULL && file.size >= 0) {
  std::string json_err_str;
  json11::Json json = json11::Json::parse((char*)file.data, json_err_str);

  int model_index = 0;
  while (json[model_index].is_object()) {
    json11::Json::object jm = json[model_index].object_items();

    Model m = {};

    auto model_name = jm["name"].string_value();
    m.name = model_name;

    auto mesh_name = jm["mesh"].string_value();
    bool found_mesh = false;
    for (int i = 0; i < asset_index; i++) {
      if (mesh_name.compare(asset_handles[i].name) == 0) {
        m.h_mesh = asset_handles[i];
        found_mesh = true;
        break;
      }
    }
    if (!found_mesh) {
      // generate asset handle
      asset_handles[asset_index] = { mesh_name, AssetType::MESH, model_buffer.num_meshes };
      // load the mesh
      asset_path = asset_folder_path;
      LoadOBJ((char*)asset_path.append(mesh_name).c_str(), &vertex_buffer, &index_buffer, &model_buffer, &frame_allocator);
      asset_path.clear();
      // assign asset handle to model
      m.h_mesh = asset_handles[asset_index++];
    }

    auto texture_name = jm["texture"].string_value();
    bool found_texture = false;
    for (int i = 0; i < asset_index; i++) {
      if (texture_name.compare(asset_handles[i].name) == 0) {
        m.h_texture = asset_handles[i];
        found_texture = true;
        break;
      }
    }
    if (!found_texture) {
      // load the texture
      asset_path = asset_folder_path;
      images[iter].data = stbi_load((char*)asset_path.append(texture_name).c_str(), &images[iter].width, &images[iter].height, &n, 4);
      asset_path.clear();
      // generate asset handle
      asset_handles[asset_index] = { texture_name.c_str(), AssetType::TEXTURE, iter };
      iter += 1;
      // assign asset handle to model
      m.h_texture = asset_handles[asset_index++];
    }

    asset_handles[asset_index++] = { model_name, AssetType::MODEL, model_index };
    models[model_index++] = m;
  }
}
```




