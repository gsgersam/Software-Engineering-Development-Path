# so_long Developer Documentation (DevDoc)

This document explains architecture, data flow, and key modules used in `so_long`.

## 1. Runtime Overview

The runtime is centered around a single `t_game` state object:

- MLX context and window handles
- Map grid and dimensions
- Player position and move count
- Texture pointers
- Validation/flood-fill support buffers

## 2. Initialization Pipeline

1. `main()` validates CLI input (`argc == 2`, `.ber` extension).
2. `start_game(map_path)` starts setup.
3. `initialize_game()` allocates and loads map data.
4. `validate_map()` verifies geometry, objects, and path reachability.
5. `mlx_init()` creates MLX context.
6. `setup_game_window()` creates the window, loads textures, and locates player.
7. Hooks are attached and `mlx_loop()` begins.

## 3. Full Flow (ASCII)

```text
[main]
  |
  +--> validate argc/extension
          |
          v
      [start_game]
          |
          v
   [initialize_game]
          |
          +--> create_and_load
          |      |
          |      +--> load_map
          |              |
          |              +--> count lines
          |              +--> validate shape
          |              +--> parse grid
          |
          +--> validate_map
          |      |
          |      +--> check_walls
          |      +--> set player
          |      +--> count objects
          |      +--> flood_fill
          |      +--> check_flood_fill
          |
          +--> mlx_init
                  |
                  v
          [setup_game_window]
                  |
                  +--> mlx_new_window
                  +--> load_textures
                  +--> init_player
                  |
                  v
             [register hooks]
                  |
                  +--> key hook (movement/ESC)
                  +--> loop hook (render on move change)
                  +--> close hook
                  |
                  v
               [mlx_loop]
                  |
                  +--> input -> update_position
                           |
                           +--> collect C
                           +--> handle E
                           +--> move_player
                           +--> redraw / move counter
                  |
                  +--> exit or win -> free_and_quit
```

## 4. Compact Flow (ASCII)

```text
main -> checks -> initialize_game
     -> load/validate map -> mlx init
     -> create window + textures
     -> hooks + event loop
     -> input -> move/render
     -> win/quit -> cleanup
```

## 5. Core Modules

- `src/main.c`: CLI checks and entry point.
- `src/game_control.c`: game startup, window setup, hook registration.
- `src/load_map.c`: file loading and map parsing.
- `src/validate_map.c`: map validity and flood-fill gate.
- `src/flood_fill.c`, `src/check_flood_fill.c`: reachability logic.
- `src/handle_input.c`, `src/update_position.c`, `src/moves.c`: player input and movement rules.
- `src/draw_map.c`: tile rendering + move counter draw.
- `src/free_and_quit.c`, `src/free_utils.c`, `src/free_textures.c`: resource cleanup.

## 6. Data Contracts

### Map tiles

- `1`: wall
- `0`: floor
- `P`: player start/current tile
- `C`: collectible
- `E`: exit

### Critical struct (`t_game`)

Holds all runtime state needed by update/render/free paths, avoiding global mutable state spread.

## 7. Error Handling Strategy

- Fail fast during loading/validation.
- Free partially allocated resources on each failure branch.
- Avoid entering the event loop when initialization is incomplete.

## 8. Build and Execution

```bash
make
./so_long maps/so_long.ber
```

### Dependencies

- `libraries/libft`
- `libraries/minilibx-linux`
- X11 libraries on Linux (`X11`, `Xext`)

## 9. Extension Points

- Bonus enemy lifecycle in `bonus/` (`enemies_init`, `enemies_update`, `enemies_free`).
- Additional animation states can be attached to the existing texture array lifecycle.
- HUD improvements can be added in `draw_moves_counter()` and render pipeline.

## 10. Known Issues / TODO

- Header hygiene: remove duplicated prototypes and standardize naming (`ft_ger*` vs `ft_ger_*`).
- Movement pipeline cleanup: keep collectible/exit handling in one place only.
- Validation diagnostics: replace generic prints with explicit failure reasons per rule.
- Memory-safety audit: verify all early-return branches release allocated resources.
- Optional: add a tiny CI-friendly map checker runner (non-graphical validation mode).
