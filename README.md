*This project has been created as part of the 42 curriculum by rmanzana, pamanzan*

# cub3D

## Description

**cub3D** is a 3D maze explorer inspired by the classic **Wolfenstein 3D** (1992), the first true first-person shooter. It renders a first-person view of a maze described in a `.cub` file, using the **raycasting** technique and the **MiniLibX** graphics library.

The goal of the project is to understand how raycasting turns a 2D grid into a 3D-looking scene, and to apply it in a real-time application: parsing and validating a scene file, loading textures, handling keyboard and mouse input, and redrawing the frame every loop iteration.

### Features

- Textured walls, with a different texture for each orientation (north, south, east, west).
- Custom floor and ceiling colours.
- Smooth movement with `W` `A` `S` `D` and rotation with the arrow keys.
- **Wall collisions**, with a screen shake effect when the player bumps into a wall.
- **Minimap** showing the area around the player.
- **Mouse rotation**, with two modes: free look (cursor hidden) or click and drag.
- On-screen controls panel.
- Strict validation of the scene file, with a clear error message for every invalid case.

## How it works

### 1. Parsing the scene file

A `.cub` file contains the texture paths, the floor and ceiling colours, and the map:

```
NO ./path/to/north_texture.xpm
SO ./path/to/south_texture.xpm
WE ./path/to/west_texture.xpm
EA ./path/to/east_texture.xpm

F 220,100,0
C 225,30,0

111111
100101
101001
1100N1
111111
```

- `1` is a wall, `0` is empty space, and `N`, `S`, `E` or `W` is the player's starting position and orientation.
- The map is read line by line and checked: every element present exactly once, valid RGB values (0–255), existing `.xpm` texture files, only valid characters, exactly one player, and a map **closed by walls** on every side (no empty space touching the outside or a space character).
- Any error prints `Error:` followed by an explicit message, and the program exits cleanly.

### 2. Raycasting with DDA

For every vertical column of the screen, a ray is cast from the player's position in the direction matching that column within the field of view.

The ray advances through the grid using the **DDA (Digital Differential Analyzer)** algorithm: instead of moving in small fixed steps, it jumps directly from one grid line to the next, choosing at each step whether the next crossing is vertical or horizontal. This finds the exact wall the ray hits with very few iterations.

Then, for each column:

- The distance to the wall is **corrected** by multiplying it by the cosine of the angle between the ray and the view direction. Using the raw ray length would cause a "fisheye" distortion, with walls bulging in the middle of the screen.
- The height of the wall slice on screen is inversely proportional to that distance: the farther the wall, the shorter the slice.
- Which side of the wall was hit (vertical or horizontal grid line, and the ray direction) decides which of the four textures to use.
- The exact point where the ray hit the wall gives the texture's **x coordinate**, and the texture is stretched vertically to fill the slice.
- Pixels above the slice get the ceiling colour, and pixels below get the floor colour.

### 3. Rendering and game loop

Every frame is drawn into an off-screen MLX image, then pushed to the window at once to avoid flickering. Key presses and releases only update the player's state (`key_up`, `left_rotate`, …), and the main loop moves the player and redraws the frame, which gives smooth, continuous movement while a key is held.

Before each move, the new position is checked against the map so the player cannot walk through walls.

## Instructions

### Requirements

The Linux version of the MiniLibX is **included** in `include/mlx_linux/` and is compiled automatically by the Makefile; nothing has to be downloaded.

The MiniLibX is built on top of X11, so the X11 development headers must be present on the system. They are already installed on the 42 computers; on a fresh Debian / Ubuntu machine:

```bash
sudo apt install libx11-dev libxext-dev
```

### Compilation

```bash
make          # builds libft, the MiniLibX and the cub3d executable
make clean    # removes object files
make fclean   # removes object files and the executable
make re       # rebuilds everything
```

### Usage

```bash
./cub3d final_maps/map.cub
```

The repository includes three folders of maps:

| Folder | Content |
|---|---|
| `final_maps/` | Polished maps with their textures |
| `maps/` | Test maps (big maps, maps with spaces, texture tests, …) |
| `wrongmaps/` | Invalid maps to test error handling (open map, missing texture, invalid colour, several players, wrong extension, …) |

### Controls

| Input | Action |
|---|---|
| `W` `A` `S` `D` | Move forward / left / backward / right |
| `←` `→` | Rotate the view |
| Mouse | Rotate the view |
| `N` | Toggle mouse mode: free look ⇄ click and drag to rotate |
| `M` | Toggle the minimap |
| `ESC` or closing the window | Quit |

## Resources

- [Lode's Computer Graphics Tutorial — Raycasting](https://lodev.org/cgtutor/raycasting.html)
- [Ray-Casting Tutorial For Game Development — F. Permadi](https://permadi.com/1996/05/ray-casting-tutorial-table-of-contents/)
- [Digital Differential Analyzer — Wikipedia](https://en.wikipedia.org/wiki/Digital_differential_analyzer_(graphics_algorithm))
- [Wolfenstein 3D — Wikipedia](https://en.wikipedia.org/wiki/Wolfenstein_3D)
- [MiniLibX documentation — harm-smits](https://harm-smits.github.io/42docs/libs/minilibx)
