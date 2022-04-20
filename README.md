## Overwatch Minesweeper

### Description
This is a classic game - [minesweeper](https://en.wikipedia.org/wiki/Minesweeper_(video_game)). In it you have to open the whole field without hitting any mines. Opening the cells you will see numbers - the number of mines around the cell. You can put a flag on a closed cell so you don't accidentally open it

### How to launch
> ### Workshop code: EJPA5

1. Create custom game \
![Create](https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/installation/01-create.jpg)
2. Press 'Settings' \
![Settings](https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/installation/02-settings.jpg)
3. Press 'Import code' \
![Import](https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/installation/03-import.jpg)
4. Insert code **EJPA5** \
![Place](https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/installation/04-place-code.jpg)
5. Enjoy

### Features
1. Press `Primary fire` to open the cell
2. Press `Secondary fire` on the closed cell to put or take away the flag
3. Press `Secondary fire` on the opened cell to open neighboring cells if number of open cell <= count of neighboring cells marked with flag
4. The field is generated after the first shot. You can be sure that the first shot always hits a cell with no mines around it
5. Fixed size of field: 10 x 10
6. Custom mine count. `Jump` in the `Red sphere` to increase the number of mines, `Crouch` to decrease
7. `Crouch` in the `Yellow sphere` to restart the game
8. Press `F` to teleport into the `Yellow sphere`
9. Press `Melee attack` to toggle top view
10. Statistics
11. You can play in co-op, but because of this can cause errors with the server or rendering elements

### Videos (youtube)
#### 1. [Game play video (Win)](https://youtu.be/xQJwK9vj_e4)
[![Game play video (win)](https://img.youtube.com/vi/xQJwK9vj_e4/0.jpg)](https://youtu.be/xQJwK9vj_e4)

#### 2. [Game play video (Lose)](https://youtu.be/MOFW_uAfurQ)
[![Game play video (lose)](https://img.youtube.com/vi/MOFW_uAfurQ/0.jpg)](https://youtu.be/MOFW_uAfurQ)

### Images
#### 1. Game not started
<img alt="game-not-started" src="https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/game-not-started.jpg" width="480">

#### 2. Game in progress
<img alt="game-in-progress" src="https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/game-in-progress.jpg" width="480">

#### 3. Game lost
<img alt="game-lost" src="https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/game-lost.jpg" width="480">

#### 4. Stats
<img alt="stats" src="https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/stats.jpg" width="480">

#### 5. Top view
<img alt="top-preview" src="https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/top-preview.jpg" width="480">

#### 6. Control panel
<img alt="controls" src="https://github.com/mnogom/overwatch-minesweeper/blob/main/examples/controls.jpg" width="480">

### How its work
#### Game states:
* `Game state not started`
* `Game state in progress`
* `Game state lost`
* `Game state win`

On `server starting`:
* Setup all global variables
* Set Current status to `Game not started`

Then `Cast rule for projectile usage`

On `game not started`:
* Create HUD hints
* Set HUD with level
* Drop lose condition
* Stop timer
* Drop flag counter
* Generate mesh grid [1]
* Create action spheres, field borders, states
* Init projectile for each player
* Switch current status to `Game not started`
* User can increase or decrease mines count
* Player open first cell
* If player hit the cell on the field => generate field. First hit always open cell with number `0`
* Switch current status to `Game in progress`

On `Game in progress`:
* Start chase timer
* Player open cell [2]
* Player toggle flag on cell
* Win condition check (`count of closed cells == mines count`) => Switch current status to `Game state win`
* Lose condition check (player touched mine) => Switch current status to `Game state lost`

On `Game state lost`:
* Stop chase timer
* Show mines
* Kill player / Increase count of losses

On `Game state win`:
* Stop timer and check personal best time
* Show message about win / Increase wins count

On `Going`
* Pressing F => teleport to Yellow sphere
* Pressing Crouch in Yellow sphere => destroy all effects/texts/flags, set current status to `Game state not started`. If game was in progress => Increase lose count
* Pressing Melee - toggle camera
* Pressing Jump/Crouch in Red sphere => increase/decrease mines count, drop statistics

Then `Destroy projctile`

**Subroutines:**
1. generateField: Get first cell -> find neighbors -> generate random mine patterns (mine position != position of first cell or neighbors)
2. getNeighbors: Find neighbors around `getNeighborsArg` and place to `getNeighborsOut`
3. findProjectileHitIndex: Find index of cell that projectile hit
4. openCell: If projectile hit cell with number != 0 -> call `_openSingleCell` otherwise call `_openStackCell`
5. _openSingleCell: Open single cell
6. _openStackCell: Open stack cells
7. flagCell: Toggle flag if cell is closed, Open neighbors if cell is opened and number of it <= count of flags around
8. showAllMines: Show all mines. Show right placed flags, Show mistake, Show all not flagged mines
9. dropStats: drop personal best time, wins count, loses count

```
[1] mesh grid - list of coordinates of the field
[2] player hit cell -> if cell is number -> open single cell (if cell is mine -> set touch mine status to True)
                    -> else -> create stack of cells -> for each -> open cell
```
### Snippets
In this project, I encountered that the text and icons start to float when the player moves. To solve this problem, I tried to recalculate the coordinates of the text and icons according to the player's position through some proportions. As a result, I got the following formula:

<img src="https://latex.codecogs.com/svg.image?\overrightarrow{V_c'}&space;=&space;\overrightarrow{V_c}&plus;\left&space;[&space;\overrightarrow{L_w}&space;\times&space;\left&space;(&space;\overrightarrow{V_c}&space;-&space;\overrightarrow{P_e}&space;\right&space;)&space;\right&space;]&space;\left&space;[&space;\dfrac{4}{50}&space;P^y_e&plus;\dfrac{1}{40}&space;\left&space;(&space;\overrightarrow{F_w}&space;\cdot&space;\left&space;(&space;\overrightarrow{V_c}&space;-&space;\overrightarrow{P_0}&space;\right&space;)&space;\right&space;)&space;\right&space;]">
where:

<span><img src="https://latex.codecogs.com/svg.image?\overrightarrow{V_c}"> - Real coordinate of text (icon)</span>

<span><img src="https://latex.codecogs.com/svg.image?\overrightarrow{L_w}"> - Left to player in world coordinates `World Vector(Left, Local Player, Rotation)`</span>

<span><img src="https://latex.codecogs.com/svg.image?\overrightarrow{P_e}"> - Eye position of local player</span>

<span><img src="https://latex.codecogs.com/svg.image?\overrightarrow{P_0}"> - Position of local player</span>

<span><img src="https://latex.codecogs.com/svg.image?P^y_e"> - Y component of player eye position</span> \
<span><img src="https://latex.codecogs.com/svg.image?\overrightarrow{F_w}"> - Forward to player in world coordinates `World Vector(Forward, Local Player, Rotation)`</span>

In workshop:
```
Create In-World Text(
    All Players(All Teams),
    Event Player._fieldCellNumber,
    Update Every Frame(
        Evaluate Once(Global.fieldCoordinates[Event Player._cellToOpen]) + Cross Product(World Vector Of(Left, Local Player, Rotation), Normalize(
        Evaluate Once(Global.fieldCoordinates[Event Player._cellToOpen]) - Eye Position(Local Player))) * (Y Component Of(Eye Position(
        Local Player)) * (4 / 50) + Dot Product(World Vector Of(Forward, Local Player, Rotation), Evaluate Once(
        Global.fieldCoordinates[Event Player._cellToOpen]) - Position Of(Local Player)) * (1 / 40))),
    2,
    Do Not Clip,
    Visible To and Position,
    Global.colorOfNumbers[Event Player._fieldCellNumber],
    Default Visibility
);
```
