---
layout: post
title:  "Archer Arena"
date:   2016-12-15
excerpt: "Final project for 15-112 Fundamentals of Programming"
project: true

comments: false
---

Final project for CMU course I took in the fall 2016 semester, 15-112 Fundamentals of Programming

{% include youtube.html id="L-PRv2UXZ3o" %}

## Survival Mode
# AI and Graphs
My term project is a 2.5d shooter where the player fights against AI. In the game there are two different sets of AI, which are defined in their own classes, though both inheriting from a parent AI class. EasyAI and MediumAI. The EasyAI's functionality relies mostly on timers and rectangle intersections. For movement it relies on a timer that changes the AI's direction ever couple seconds. In order for the EasyAI to avoid running into obstacles, it has a rectangle created in front of it that detects an intersection with a wall. When such an intersection occurs, the EasyAI skips the timer and changes direction. In this way the EasyAI wanders around the map randomly until, by a similar process using rectangles, finds the player and begins shooting arrows. To avoid shooting unnecessarily, the AI checks to see if there is a wall in between itself and the player. If there is, it won’t shoot. The EasyAI has one health and will die when hit by an arrow

# Graphs
The MediumAI, rather than wandering around aimlessly, uses a generated graph and Breadth First Search to seek out the player. In this process the MediumAI first sees if it can attack the player, using the same rectangle process as the EasyAI. If it is not in range, then it follows the path set up by the BFS method. The BFS method uses graph theory to determine the shortest route to the player while also considering obstacles. This is accomplished by generating a graph that has repeated nodes layered over the map. Nodes are placed in a regular order and are not placed when they lie within a obstacle. Additionally, edges are formed between nodes that are adjacent to each other in the x and y direction. Through this graph the MediumAI can use the player’s position on the graph and then its own position to determine the best path and make a move. It repeats this process of movement and attack until the player is dead. The only break in this process is when the player shoots an arrow towards the MediumAI. The MediumAI has a set of rectangles that lie in all 4 directions that will detect a player’s arrow if it intersects with one of them. First the “dangerDetector” determines if the arrow is headed towards the mediumAI. If it is, then the MediumAI will do its best to dodge the arrow by moving out of the way. Avoiding danger takes priority over pursuing the player and attacking and will be done first. The MediumAI has two health and will lose part of its armor when hit by an arrow. Note: By pressing g during survival, the map’s graph is displayed. By pressing s, the path of the mediumAI is shown.

<figure>
  <a href="/assets/img/archerarena/AAgraph.png"><img src="/assets/img/archerarena/AAgraph.png"></a>
  <figcaption>Generated Graph for Basic Map</figcaption>
</figure>

# Character
Character is a class that both the Player and AI’s inherit that handles all animations through a series of clipping sprite sheets to their appropriate sections and cycling through these clippings to create movement and animation. Additionally the function updateWalk checks if the character is running into a wall and will prohibit the character from passing through.

# Potions and Projectiles
Projectiles are sprites and are the arrows that the player and AI shoot at each other. They have an initial direction given to them by the direction that the player or AI shoots from and then they update and move across the screen until they collide with a wall or character. Potions are for the player to pick up only. They provide health to the player but only when the player is below maximum health. Within the main game class a location that is not within a wall is randomly chosen and then the health potion is spawned.

# Player
The player class handles only the init of the character and the update of its movements and graphics. Within the game class is the spawning of the player. By default the player will spawn in the top left of the screen. But, in custom map mode, the user can decide to cover up the top left of the screen. Then, the spawning system will seek out the next open place on the map and spawn the player there. The player’s actions are controlled by the key events. Where movement is controlled by the arrow keys, spacebar to shoot.

<figure>
 <a href="/assets/img/archerarena/AAplayerarrow.png"><img src="/assets/img/archerarena/AAplayerarrow.png"></a>
 <figcaption>Sprite Sheet for Player Animation</figcaption>
</figure>

# Game Methods
If there are no more enemies left on the screen, the game spawns the next wave of enemies. By default these enemies will spawn in the quadrant that is opposite to the player. So if the player is in the bottom right quadrant when the round ends, the enemies will spawn in the top left. If this area is covered, the AI will spawn in the next quadrant in clockwise order. As the number of waves that the player has survived increases, the number of enemies that spawn in each wave increases as well. Additionally, the likelihood that each spawned enemy is a mediumAI is increased as well.

Timerfired calls multiple different update functions on the groups that contain the player, enemies, and arrows. Within these the AI process will be run and updated to be drawn on the screen. Additionally, the methods check for collisions between the player and arrows, and the enemies and arrows. This will result in the loss of health for players and AI when struck and also the removal of arrows from their groups as they collide. A similar process is done with arrows and obstacles so that the arrows do not pass through walls.

There are multiple draw functions that draw out the player, the enemies, arrows, other sprites, walls, etc. Included also are ones that will display the graph and BFSPaths.

# Survival Mode UI
The User Experience here is partially inspired by the player vs player that I found in Castle Crashers. Though castle crashers is more dimensional than this game, as it allows jumping and use of shields to block arrows. Instead I have chosen to keep the core versus features of the game, but instead replacing the PvP game mode with a player vs AI. There are obstacles to hide behind and to avoid enemies like in the PvP mode of castle crashers and the player has health and he will die if he runs out. Another inspiration for the UI was the indie developed game Secrets of Grindea. Graphics wise, I searched for sprite sheets that strongly resembled that game because I thought it would blend well with the type of model that I was looking for. Secrets of Grindea though for the most part is a long RPG game with multiple maps and this is far beyond the scale that I was looking for so I instead drew from the fighting aspect of the game. From both Castle Crashers and Secrets of Grindea, I had to remove melee combat because both games utilize shields to block attacks and I chose to focus on the archery aspect instead. The controls for the game are inspired by what has been always traditionally done for games like this. Secrets of Grindea uses the arrow keys to move and ‘z’ to attack but I instead chose to have the spacebar to attack.

# Custom Levels
From the splash screen, the custom levels mode can be access by selecting “Custom Level.” The custom levels mode can also be reached by pressing “e” while in the survival mode. The custom levels mode can back up into the menu screen by pressing “escape” and can return to the survival mode by pressing “p”

The way the custom levels mode works is that is starts with a list of obstacles (Pygame Rectangles) and also a 2d list representation of the board, both by default empty (Depends if one is coming fresh from the menu or from the survival mode).

On the display is a faded wallPiece, representing that if the mouse was pressed, a wall would be placed in that location. The user can click and drag on the screen to create a desired rectangular wall and place it on the map. This is accomplished by storing the start and end points of the click and drag, converting it to the grid format, and updating the model to later be drawn. Implemented also is an undo and redo feature that will undo previous mistakes and also redo mistaken mistakes. (r key for redo, u key for undo). The redo and undo store the recent 2d representations and also the deleted or redone rectangles. Additionally, when the user tries to place a wall, the system checks if every open point on the board is then reachable. If it cannot reach every point, then it will not allow the placement of the wall. This is what requires the 2d representation of the map in addition to the pygame representation.

<figure>
  <a href="/assets/img/archerarena/AAcustom.png"><img src="/assets/img/archerarena/AAcustom.png"></a>
  <figcaption>Example of Creating a Custom Map</figcaption>
</figure>

# Custom Level UI
Although Secrets of Grindea and Castle Crashers both don’t have a custom level system in their game. I thought it would benefit the user experience to enable the user to create their own maps. From most custom level portions of games that I have seen, the user is usually able to visually see the addition he or she is about to make. For this reason I have the faded block displayed for whenever the player is hovering over the screen or clicking and dragging out. The user also tends to make mistakes when creating a map, so I also implemented an undo and redo feature for them, so they don’t have to start over when they make a bad placement. Additionally, the game checks that there are not too many walls on the screen, in order to give the game room to spawn characters. The user is notified on how much space they can take up, which is 2/3 of the screen space. If this was not in place then the user experience in the survival mode would suffer.

# Splash Screen
The splash screen is the main menu of the game. From it the user can reach survival mode, custom map mode and the help screen.

# Help
The help screen is a displayed png image that gives the players instruction on how to play survival.

On the screen is an MediumAI vs EasyAI scenario. They both treat the other as they would the player. The EasyAI wanders and attacks if the MediumAI is in range, and then the MediumAI seeks out the EasyAI and attacks the EasyAI and dodges the EasyAI’s arrows. When either the EasyAI or MediumAI dies, they are respawned.

## Citations
# Languages

Python

# Graphics

[Characters](http://opengameart.org/content/lpc-medieval-fantasy-character-sprites)

[Walls](http://opengameart.org/content/16x16-pixel-art-dungeon-wall-and-cobblestone-floor-tiles)

[Potion](http://opengameart.org/content/potion-0)

[Arrow Keys](http://www.101computing.net/wp/wp-content/uploads/arrowKeys.png)

[Space bar](http://www.joshbenson.com/geeklift-spacebar-trick-google-power-search-and-ccp-hotkeys/space-bar-trick/)

[P, E, G, S, Escape Key](http://www.wpclipart.com/computer/keyboard_keys/)

Sprite Sheet Clipping Adapted from [here](http://xorobabel.blogspot.com/2013/01/an-updated-guide-to-implementing-2d.html)

Inspiration for Breadth First Search are from The 112 Optional Lecture on Graph Theory

Pygamegame.py from Lukas Peraza 112 Lecture on pygame

FloodFill taken from 112 [website](http://www.cs.cmu.edu/~112/notes/notes-recursion-part2.html#floodFill)
