# cub3D

A group project involving a Wolfenstein-inspired raycasting engine built with MiniLibX as part of the 42 curriculum. The project features a clear split between the map parser (handling configuration and validation) and the rendering engine (calculating wall intersections and textures).**mandatory** and **bonus** gameplay.

## What this project includes

### Mandatory (`cub3D`)
- `.cub` parsing and strict map validation
- Wall raycasting with textured north/south/east/west faces
- Floor/ceiling colors
- Player movement (`W`, `A`, `S`, `D`) and camera rotation (arrow keys)
- Real-time frame rendering using MiniLibX

### Bonus (`cub3D_bonus`)
- Doors (`D`) with interaction (`E`)
- Sprites and enemies (`2`, `M`)
- HUD (health, ammo, crosshair, weapon label)
- Minimap
- Mouse-look camera rotation
- Weapon system (switch with `1` / `2`, shoot with `Space` and mouse)
- Audio integration (SDL2_mixer)

## Requirements

- Linux (X11)
- `make`, `cc`
- SDL2 + SDL2_mixer + `sdl2-config`
- MiniLibX (already included in this repository under `mlx/`)

> Note: the Makefile uses `SDL_MIXER_DIR := $(HOME)/SDL2_mixer`. If your SDL2_mixer lives elsewhere, update this path.

## Build

```bash
make
```

Build bonus:

```bash
make bonus
```

Debug rebuild:

```bash
make debug
```

Clean:

```bash
make clean
make fclean
```

## Run

Mandatory:

```bash
./cub3D maps_m/m0.cub
```

Bonus:

```bash
./cub3D_bonus maps_b/bm0.cub
```

## Map format (`.cub`)

### Header entries (required)
- `NO <path>` north texture
- `SO <path>` south texture
- `WE <path>` west texture
- `EA <path>` east texture
- `F R,G,B` floor color
- `C R,G,B` ceiling color

### Map symbols
- Mandatory: `1`, `0`, `N`, `S`, `E`, `W`, ` `
- Bonus additions: `D` (door), `2` (sprite), `M` (monster)

The map must be enclosed by walls and contain exactly one player start.

## Controls

### Mandatory
- Move: `W A S D`
- Rotate: `Left Arrow` / `Right Arrow`
- Quit: `Esc`

### Bonus
- Interact with doors: `E`
- Shoot: `Space` and left mouse button
- Sprint: `Shift`
- Weapon switch: `1`, `2`
- Mouse-look: move mouse

## ASCII flow

### Startup flow

```text
+-------------------+
| Program Start     |
+-------------------+
        |
        v
+-------------------+
| Check args (.cub) |
+-------------------+
        |
        v
+-------------------------------+
| Parse header: NO SO WE EA F C |
+-------------------------------+
        |
        v
+---------------------------------------------+
| Parse map + set player + validate enclosure |
+---------------------------------------------+
        |
        v
+--------------------------------+
| Init MLX + textures + z-buffer |
+--------------------------------+
        |
        v
    +----+---------------------+
    | Build type?              |
    +----+---------------------+
        |                     |
        | Mandatory           | Bonus
        v                     v
 +-------------------+   +----------------------------------+
 | Register base     |   | Init bonus objects/resources:    |
 | hooks             |   | doors, sprites, guns, UI, audio  |
 +-------------------+   +----------------------------------+
        |                     |
        +----------+----------+
                |
                v
        +----------------------+
        | Enter mlx_loop       |
        +----------------------+
```

### Frame loop (Mandatory)

```text
[loop_hook]
   |
   v
update delta_time
   |
   v
move_player
   |
   v
cast_rays
   |
   v
draw floor/ceiling + walls
   |
   v
mlx_put_image_to_window
```

### Frame loop (Bonus)

```text
[loop_hook]
   |
   v
update delta_time
   |
   v
process_input (shoot/interaction requests)
   |
   v
update_bonus_logic
  - doors (toggle + cooldown)
  - gun animation + muzzle flash
  - sprites/enemy states
   |
   v
move_player + cast_rays
   |
   v
draw world
   |
   v
render_bonus_ui
  - minimap
  - sprites
  - gun + HUD
  - text prompts
```

## Repository layout

- `src/` mandatory implementation
- `bonus_src/` bonus-specific modules
- `includes/` headers for shared and bonus APIs
- `maps_m/` mandatory maps
- `maps_b/` bonus maps
- `textures_m/` mandatory textures
- `bonus_textures/` bonus textures
- `sounds/` audio assets
- `libft/` local libft dependency
- `mlx/` MiniLibX source

## Notes for evaluators and contributors

- `make` builds `cub3D` (mandatory target)
- `make bonus` builds `cub3D_bonus` with `-DBONUS`
- Bonus reuses mandatory sources where not overridden (fallback object rules in Makefile)

Developed in collaboration with ![Ali Rahmouned](https://profile.intra.42.fr/users/alrahmou) ![German Andres Sanin](https://github.com/gsgersam)
