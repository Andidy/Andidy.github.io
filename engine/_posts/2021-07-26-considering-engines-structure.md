---
layout: post
title: \[Engine\] Considering engine's structure before moving forward
---

Currently engine’s code is structured in a way similar to the way object-oriented classes are defined.
The program is broken up into 3 main modules or layers, the platform layer, the rendering layer, and the game layer.
The idea behind this structure is that each layer’s primary dependency is the previous layer.
The platform layer is the main entry point and cannot function without the specific OS its built for.
The renderer cannot function without the platform layer and any rendering API the OS may provide.
The game cannot function without the platform layer to provide input, sound and display,
and the rendering layer to calculate the image to provide to the display.
Each layer contains the data declarations and definitions of its classes, structs and functions.
For example: the game has a struct “GameState” which stores things like game object positions and rotations
and the position and rotation of the game’s camera and contains functions like “GameUpdate” which updates
the game state based on the player’s input, the renderer contains data like the vertex buffers and shader references
to know what it needs to draw to the screen, and functions like “RenderFrame” which make the appropriate API calls to draw the frame,
and similar for the platform layer.

![three_layers](/docs/assets/images/blog/considering-engines-structure/three_layers.png)

Theoretically, this structure works well and provides a way for the user to swap out one or more of the layers to achieve a desired result.
A port to Linux, PlayStation, MacOS, or mobile devices theoretically only requires replacing the platform layer,
assuming a compatible rendering layer is present. An upgrade to a new fancy graphics API, or just switching to support a different OS,
only requires replacing the rendering layer. To make a different game, only replacing the game layer is required.

![swappable_layers](/docs/assets/images/blog/considering-engines-structure/swappable_layers.png)

In actuality I’m not sure anymore that this works. I’ll give two examples: Game Input and Object Picking.
So, let’s imagine you’ve created the 3 layers and you’ve decided to start working with a 3D game, D3D11, and Win32, for example.
Very quickly, as you are working on your project you will need user input for the game in order to test your code and *provide the gameplay *.
So you go to the platform layer, you look up the Win32 APIs for getting keyboard and mouse input and controller input, and then you implement them.
Great! Problem is, how are you supposed to get the input from the user? After all, the game isn’t allowed to know anything about the platform layer,
because otherwise how would you be able to swap the platform layer out without re-writing the game code? It defeats the purpose of the separation.
You can imagine other situations where this might occur: How does the game tell the platform layer to play the background music?
Or to load the assets of the next level? Or to tell the renderer that “The player is badly hurt, apply the bloody screen effect”?

My solution to this was to acknowledge that somethings will need to be accessible to “higher” layers in the stack.
The logical step from that is to move the declarations of those things to a point higher in the stack than where they are required.
This way the things that need to access lower layers can do so in those special cases. I called this new layer the “universal” layer,
because I include it in the platform, renderer, and game layers.

![universal_layer](/docs/assets/images/blog/considering-engines-structure/universal_layer.png)

With the universal layer, we can move some declarations of our classes and structs and functions into it so that the higher level
game and renderer layers can access lower level dependencies. For example: We can create a struct called Input which contains the keyboard and mouse state,
initialize it and store it in the memory of the platform layer, and then every iteration of our main loop
we can call the platform specific API to update the state for the next loop iteration, but also pass a pointer to it into the game update function
so that the game can read the state of the input peripherals and do checks like “if keyDown(“A”) move_player_left();” or “if keyPressed(“P”) pause_game();”.

I’ve broken the complete separation of the higher layers from the lower layers at this point,
though out of necessity as otherwise it wouldn’t be much of a game if there is no user input.
The new universal layer in theory isn’t too bad as long as the user keeps it from relying on the way a specific module works.
For example: the universal layer stops being universal if it contains a struct with a LARGE_INTEGER typed field because that is a Win32 specific type,
or it shouldn’t contain a reference to an ID3D11Buffer since then it would be reliant on the D3D11 library and API.

Now, let’s discuss object picking. In order to select an object displayed in the game world it is necessary to convert from the screen space,
defined as the 2D location of the mouse cursor, into the world space of the scene so that we can cast a ray and do intersection checks
against the physics colliders or rendered meshes in order to select a specific object in the game world.
An example of where this is used would be in a strategy game, in order to use the mouse to select a unit or click on a grid position to get information about it,
this is a place where object picking is needed. The reason this is an issue for the current way I have structured the code in engine is that
object picking relies heavily on data that so far has been “owned” by the renderer: FOV, viewport width and height,
and in viewports that don’t fill the entire region also require the max and min z depth and the x and y positions of the viewport.
So now I have to decide if I should create a number of functions which somehow get this data from the renderer,
which up until now the game only ever sent data to (through the platform layer), never received data from or,
if I should restructure the code into a new structure which better fits the new problem space I’ve discovered with continued development.