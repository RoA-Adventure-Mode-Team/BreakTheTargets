# When you Break Into Target I dunno I never played the game mode
Here's a comprehensive guide to making your own Break the Target courses for the Break the Targets minigame on the RoA Workshop. It uses an early version of the RoA Adventure Mode API, meaning it has a lot of support for features beyond the minigame. But, we will only discuss things which are relevant to making a custom Break the Targets course. I will write a guide to the API once it and the example level is finished. In the meantime, have fun with this version tailored for this minigame.

## Required Code
Here's the required snippets of code before you custom code anything.

In `init.gml`:
```
get_btt_data = false;
```
having `get_btt_data` as an existing variable is how the stage knows when to read a character's custom course.

In `update.gml` **at the end of the script**:
```
#define room_add(_room_id,room_data) //Adds a new room to the scene
with obj_stage_article if num == 5 {
	var _room_id_ind = array_find_index(array_room_ID,_room_id);
	if _room_id_ind == - 1 {
	    if debug print_debug("[RM] Adding... "+string(_room_id));
	    array_push(array_room_data,room_data);
	    array_push(array_room_ID,_room_id);
	} else {
	    array_room_data[_room_id_ind] = room_data;
	    array_room_ID[_room_id_ind] = _room_id;
	}
}
```
This function adds rooms to the stage's room array. You can have multiple rooms, but for the basics you don't need to know more than that this is how the stage keeps data.

## RoA AM API Structure
The RoA AM API is built to handle infinitely large worlds. It does so by breaking it up into "tiles", "cells", and "rooms." Tiles are 16x16 pixels large and are the main unit used to place articles in the world. Cells are by default 163x85 tiles (the max size of the stage editor).  Cells are stitched together seamlessly, but only the cell you are currently occupying and immediately adjacent cells are loaded at any time. Only 1 room at a time can be loaded, and each room contains the data for its cells.

## Formatting Overview 
Here's R-00's custom course code, put in `update.gml`:
```
if get_btt_data { //Get data for Break The Targets
	course_name = "R-00 Course";
	//Set the spawn properties
	respawn_point = [[29,50],[0,0],1];
	//Set the collision of the solid sprites to precise
	sprite_change_collision_mask("btt_solid",true, 0, 0, 0, 0, 0, 0 );  
	room_add(1,[
	    [ //Each Cell
	        [0,0], //Cell Coordinates
	        [
	        	//Targets
		        [10, 4, 55, 0, -5, [0, 0, 32, [[0,0],[0,-3]], 0, 0, 0, 0], [0]],
		        [10, 40, 30.5, 0, -5, [1, 0, 60, [[-10,0],[5,0]], 0, 0, 0, 0], [0]],
		        [10, 87, 46, 0, -5, [2, 0, 0, 0, 0, 0, 0, 0], [0]],
		        [10, 52, 44, 0, -5, [3, 0, 0, 0, 0, 0, 0, 0], [0]],
		        [10, 55, 75, 0, -5, [3, 0, 0, 0, 0, 0, 0, 0], [0]],
		        [10, 125, 55, 0, -5, [4, 0, 32, [[0,0],[0,-1]], 0, 0, 0, 0], [0]],
		        //Solid Ground
		    	[1, 2, 2, 2, 0, [sprite_get("btt_solid"), 0, 0, 0, 0, 0, 0, 0], [0]],
		    	//Plats
		    	[1, 46, 49, 1, 0, [sprite_get("btt_plat_64"), 0, 0, 0, 0, 0, 0, 0], [0]],
		    	[1, 64, 71, 1, 0, [sprite_get("btt_plat_64"), 0, 0, 0, 0, 0, 0, 0], [0]]
	            ]
	        ],
	    //Blastzones
	    [ //Each Cell
	        [0,1], //Cell Coordinates
	        [
	            [4, 0, 32, 0, 0, [4, 0, 0, 0, 0, 163*16, 20, 0], [0,0]]
	            ]
	        ],
	    [
	        [1,1],
	        [
	        	[4, 0, 32, 0, 0, [4, 0, 0, 0, 0, 163*16, 20, 0], [0,0]]
	            ]
	        ],
	    [ //Each Cell
	        [-1,1], //Cell Coordinates
	        [
	        	[4, 0, 32, 0, 0, [4, 0, 0, 0, 0, 163*16, 20, 0], [0,0]]
	            ]
	        ]
	    ]);
}

```
I highly recommend you follow this format as close as possible, as it is a crazy nested array with a lot of moving parts. Let's break it down further.


### Spawn and Solid Collision Properties
```
...
course_name = "R-00 Course";
//Set the spawn properties
respawn_point = [[29,50],[0,0],1];
//Set the collision of the solid sprites to precise
sprite_change_collision_mask("btt_solid",true, 0, 0, 0, 0, 0, 0 );  
...
```
`course_name` is the name that is displayed at the bottom left corner.
The spawn location is of the format:
`respawn_point = [[tile_x,tile_y],[cell_x,cell_y],room_id];`
This is the starting spot the player will spawn at.

The third bit changes the collision mask of the solid article sheet to be continuous - meaning it only applies collision to pixels that are not transparent. It allows us to consolidate all solid articles into 1, saving a lot of resources. Keep it as the above formatting, just changing the first string to be your course sprite's name.

### Spawning Articles
`room_add` has the following format:
```
room_add(room_id, [
	[ //Each Cell
        [0,0], //Cell Coordinates
        [
        	//Articles
	        [article_num, x, y, article_type, depth, [article_specific_args], [spawn_flag]],
	        .
	        .
	        .
            ]
        ],
    .
    .
    .
    ]);
```
Each cell is an array entry defined in two parts - the first being the cell coordinates that defines where it should be placed in the room, and the list of articles that spawn in it. **Article x & y values are relative to the top left corner of the cell it's in.**
`article_num` determines the article's `num`, aka what type it spawns as.
`article_type` determines whether or not it's a solid (2), platform (1), or no collision (0) article.
`depth` determines draw order and if it's in front or behind other articles. (positive meaning further back)
`article_specific_args` are specific to each article type, explained further bellow.
`spawn_flag` is used to track spawning. Keep at 0.
### Article Types
In this version of the AM API, there are a few different article types, ordered by article number:
1. Terrain (1)
2. Empty (2)
3. Scene Manager (3)
4. Trigger Zone/Blast Zone (4)
5. Room Manager (5)
6. Empty (6)
7. Camera Controller (7)
8. Room Transition (8)
9. Checkpoint (9)
10. Target (10)

The most important 3 you need to know are Terrain, Trigger Zones, and Targets. The others are perfectly functioning, but will go over them in the advanced section. (aka when AM is done.)

#### Terrain (1)
The terrain article has the following argument format:

`[1, x, y, article_type, depth, [sprite_index, animation_speed, 0, 0, 0, 0, 0, 0], [spawn_flag]];`

Note: `article_type` here should be 1 (platform) or 2 (solid) to enable collision.

`sprite_index` is the sprite index used for the terrain article. NOTE: sprites should be 1x1 scale and not sized up, as the API does this in game.

`animation_speed` if your article has animation, set the speed here.

#### Trigger Zone (4)
The Trigger Zone article in BtT is used to make blast zones. It has the following argument format:

`[4, x, y, article_type, depth, [event_id, active_scene, trigger_obj_type, trigger_player, trigger_shape, trigger_w, trigger_h, trigger_negative], [spawn_flag]];`

`event_id` keep as 4. It relates to calling custom event code, and for BtT event 4 is to act as a blast zone.

`active_scene` keep at 0. If not zero, the trigger will only occur on a specific scene. If it's zero, it's active no matter the scene (scenes will not be explained here).

`trigger_obj_type` keep at 0, 0 means it activates from players entering it.

`trigger_player` keep at 0, 0 means it activates no matter the player slot.

`trigger_shape`determines the shape of the trigger. 0 is rectangle, 1 is circle, and 2 uses the sprite_index. (default is 0)

`trigger_w` is the width/radius of the trigger zone (depending on the shape)

`trigger_h` is the height of the trigger zone if a rectangle.

`trigger_negative` keep 0, triggers unless inside. Experimental, might not work.

#### Target (10)
Targets are destroyed when hit with hitboxes. They have the following argument format:

`[10, x, y, article_type, depth, [targ_id, event_id, move_time, path, 0, 0, 0, 0], `

`targ_id` is the target's id. Used for debugging positions when creating a course.

`event_id` triggers the event_id when destroyed. Keep at event 0, which for BtT is the target destroy event.

`move_time` if the target has a defined path, takes this much time between points.

`path` has the format of: `[[x0,y0],[x1,y1]...]` and defines the path a target will take relative to its starting position and will loop back to the first point once it hits the end of the array.

### Debug Console
As a final note, since this is the AM API you have access to the debug console by pressing **`** on your keyboard and typing commands with it. Type 'help' for the list of implemented commands. You submit commands with attack since keyboard_string does not detect enter inputs.
*NOTE: 'debug' is something that is really handy.*

### Aether Variants
Aether variants of the BtT stage put a time cap on the level instead of a ticking up timer. You can set the maximum timer with the following line of code:
`with obj_stage_article if num == 5  max_timer = XXX`
It defaults to 300 frames.

## How to Make a Course, Actually.
Now that we have the dictionary above defining each of the parts, we can detail a step by step process.

### 1: Make Your Solid Collision Article Sprites
It's best to do this first so that you have a good basic layout to place articles on. Here's what R-00's looks like:
![R-00 Solid Sprite](https://i.imgur.com/8vuJJJz.png)

It should have all of your static, solid collisions you plan on having in a cell. It has dimensions of 1336x380 due to the cell's size being 163x85. 
*NOTE: MOST cases should only need 1 cell, and 1 sprite.*

These are the sprites that will be specially designated with `sprite_change_collision_mask` and should be placed at 2,2 in their respective cell.

In the BtT folder there's a basic 16 tileset of the training mode stage.

### 2: Place Articles
We unfortunately do not have an automatic way to pull locations reliably 1-to-1. You can use your solid collision sprites to get a rough idea of where you should place your targets (tiles are pixels/16, and remove 2 tiles due to offsets), then use the in-game console and debug mode to fine-tune the locations of the articles. The red circle is the camera position, and is detailed in the upper left corner of the debug info.

See the above section on how each of the articles are defined individually, and the example format. I highly recommend you stick with that since it can get messy quick if not careful.

You can also reference the format used in `user_event1.gml` inside the BtT scripts. 


## Questions or Troubleshooting
If you have any questions feel free to ask in the troubleshooting section of the AM API discord (https://discord.gg/FPFPJrk), or DM me ExW | Archytas#4534 !
