# so_long

A small 2D top-down game built with **MiniLibX** for the 42 `so_long` project.

The player must collect all collectibles and then reach the exit.

## Features

- `.ber` map loading and validation
- Wall-closed and rectangular map checks
- Required object checks (`P`, `E`, `C`)
- Reachability validation using flood fill
- Real-time movement and move counter
- Graceful cleanup of MLX resources

## Build

```bash
make
```

## Run

```bash
./so_long maps/so_long.ber
```

## Controls

- `W` / `↑`: move up
- `S` / `↓`: move down
- `A` / `←`: move left
- `D` / `→`: move right
- `ESC`: quit

## Full Flow (ASCII)

```text
+-------------------+
| main(argc, argv)  |
+-------------------+
          |
          v
+---------------------------+
| Check argc == 2           |
| Check extension == .ber   |
+---------------------------+
          |
          v
+---------------------------+
| start_game(map_path)      |
+---------------------------+
          |
          v
+---------------------------+
| initialize_game()         |
| - create_and_load()       |
| - load_map()              |
| - validate_map()          |
| - mlx_init()              |
+---------------------------+
          |
          v
+---------------------------+
| setup_game_window()       |
| - mlx_new_window()        |
| - load_textures()         |
| - init_player()           |
+---------------------------+
          |
          v
+---------------------------+
| Register hooks            |
| - mlx_key_hook            |
| - mlx_loop_hook           |
| - window close hook       |
+---------------------------+
          |
          v
+---------------------------+
| mlx_loop()                |
+---------------------------+
          |
          v
+---------------------------+
| Input -> update_position  |
| -> move_player            |
| -> draw_map / render_game |
+---------------------------+
          |
          v
+---------------------------+
| Win condition             |
| collectables == 0 && exit |
+---------------------------+
          |
          v
+---------------------------+
| free_and_quit()           |
+---------------------------+
```

## Compact Flow (ASCII)

```text
main -> arg/ext check -> start_game
     -> load_map + validate_map + mlx_init
     -> setup window + textures + player
     -> hooks + mlx_loop
     -> input -> movement/render
     -> win or esc -> cleanup
```

## Map Rules

- File extension must be `.ber`
- Map must be rectangular
- Map must be surrounded by walls (`1`)
- Exactly one player (`P`)
- At least one exit (`E`)
- At least one collectible (`C`)
- A valid path must exist to collectibles and exit

## Project Structure

```text
includes/     headers (`so_long.h`)
src/          game logic and rendering
bonus/        bonus enemy modules
maps/         test and playable maps
libraries/    libft + minilibx-linux
textures/     xpm assets
```

## Notes

- Build uses `gcc` + MiniLibX (`-lmlx -lX11 -lXext`).
- Linux/X11 environment is required for running the game window.

## Known Issues / TODO

- Remove duplicate declarations and naming inconsistencies in `includes/so_long.h`.
- Consolidate movement flow to avoid duplicated collectible/exit checks.
- Improve error messages with consistent `Error\n<reason>` formatting.
- Add a basic automated map-validation test set (script-based smoke tests).
- Review texture index constants vs array size usage to prevent mismatches.
