---
layout: page
title: frontier (Inactive)
---

Frontier is a software rendered 2D city building game. [GitHub](github.com/Andidy/frontier)

![An image of the game with various buildings constructed](/docs/assets/images/frontier/buildings.png)

It features a number of building types all moddable by accessing the building template JSON file. 
Buildings can produce resources and consume resources during the process to create production chains
(Wheat Farm -> Mill -> Bakery and Orchard -> Brewery being examples). An example building template for the wheat field:

```json
{
      "_name": "Field",
      "ENUM": 2,
      "RESOURCE_INPUT": [
        [ 0, 0 ]
      ],
      "RESOURCE_OUTPUT": [
        [ 1, 5 ]
      ],
      "PRODUCTION_TICKS": 10,
      "RESOURCES_COST_TO_BUILD": [ [ 4, 1 ] ],
      "TICKS_TO_BUILD": 5
}
```

There are units which can be selected and moved around the map with a basic pathfinding algorithm. The units have a basic
attack and can be ordered to damage another unit when next to it.

![an image of basic pathfinding for a unit.](/docs/assets/images/frontier/basic_pathing.png)

The UI and game map are rendered seperately with the game map supporting panning and scaling from 1x to 8x. Animation is
supported. The game map rendering is tile based and so in order to improve visual fidelity I implemented Wang-Blob tiles 
so that borders between different terrain types would have interesting graphics. I also implemented sub-tile variance using
blue noise so that even when two adjacent tiles have the same Wang-Blob tile type they are likely to have a different set of
the four sub-tiles that make up a full game map tile. Compare the grass textures in the following:

![An image showing grass and water without graphical improvements.](/docs/assets/images/frontier/basic_graphics.png)
![An image showing how Wang Blob tiles improves the look of terrain borders.](/docs/assets/images/frontier/wang_blob_tiles.png)
![An image of Wang Blob with sub tile variants for maximum fidelity.](/docs/assets/images/frontier/wang_blob_with_variants.png)

Frontier features keyboard and mouse support and a basic UI with simple buttons, panels, and bitmap based text.

Frontier is inactive because I am more interested in 3D graphics (See [Engine](https://andidy.github.io/engine)) right now.

[Back to homepage](https://andidy.github.io/)