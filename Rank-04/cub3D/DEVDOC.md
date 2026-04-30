# cub3D Developer Documentation

This document explains the internal architecture, execution flow, and how mandatory and bonus builds differ.

## 1) Build model and target split

- Mandatory binary: `cub3D`
- Bonus binary: `cub3D_bonus`
- Compile-time feature switch: `-DBONUS`

The Makefile builds:
- mandatory objects from `src/`
- bonus objects from `bonus_src/`
- plus mandatory fallback objects when bonus does not override a module

This keeps shared logic in one place and bonus logic modular.

## 2) Startup pipeline

`main()` zeroes `t_data`, validates args, calls `init_game()`, then `run_game()`.

### Startup ASCII flow (function-level)

```text
+------------------------------+
| main(argc, argv)             |
+------------------------------+
    |
    +--> ft_bzero(&data)
    +--> argc != 2 ? print usage + EXIT_FAILURE
    |
    v
+-------------------------------------------+
| init_game(&data, argv[1])                 |
+-------------------------------------------+
    |
    +--> check_extension(map_path)
    +--> parse_header(data, map_path)
    +--> get_map_dimensions(data, &map, path)
    +--> parse_map(data, &map, path)
    +--> set_player_position(data)
    +--> validate_map(data)
    +--> [BONUS] init_bonus_game_objects(data)
    +--> init_mlx(data)
    +--> init_images(data)
    +--> set_img_info(&main_img, ...)
    +--> init_resources(data)
    |       +--> load_textures(data)
    |       +--> [BONUS] init_bonus_resources(data)
    |       +--> z_buffer = ft_calloc(...)
    +--> data.last_frame_time = get_time_in_ms()
    +--> set_bonus_defaults(data)
    |
    v
+------------------------------+
| run_game(&data)              |
+------------------------------+
    |
    +--> [MAND] mlx_hook(..., handle_keypress/handle_keyrelease)
    +--> [BONUS] mlx_hook(..., main_keypress/main_keyrelease/mouse_move)
    +--> mlx_loop_hook(..., render_frame_with_delta)
    +--> mlx_loop(data.mlx)
    |
    v
+------------------------------+
| free_game_resources(&data)   |
+------------------------------+
```

## 3) Main loop and frame processing

Both builds use `mlx_loop_hook()` with frame timing (`delta_time`) clamped for stability.

### Mandatory frame flow (function-level)

```text
mlx_loop_hook -> render_frame_with_delta(data)
        |
        +--> update_last_frame_time(data, &now, &delta_time)
        +--> render_frame(data, delta_time)
                 |
                 +--> draw_floor_ceiling(data)
                 +--> move_player(data, delta_time)
                 +--> cast_rays(data)
                 +--> mlx_put_image_to_window(..., data->screen.img, ...)
```

### Bonus frame flow (function-level)

```text
mlx_loop_hook -> render_frame_with_delta(data)
        |
        +--> update_last_frame_time(data, &now, &delta_time)
        +--> process_input(data)
        +--> update_bonus_logic(data)
        |      +--> handle_door_logic(data, now)
        |      +--> handle_gun_logic(data, now)
        |      +--> handle_sprite_logic(data, now)
        |      +--> bonus movement collision checks
        +--> render_frame(data, delta_time)
               |
               +--> draw_floor_ceiling(data)
               +--> move_player(data, delta_time)
               +--> cast_rays(data)
               +--> mlx_put_image_to_window(..., data->screen.img, ...)
               +--> render_bonus_ui(data)
                      +--> render_minimap(data)
                      +--> render_sprites(data)
                      +--> draw_gun(data)
                      +--> render_health_bar/render_ammo_bar/draw_crosshair
```

## 4) Input system

### Mandatory
- `handle_keypress` / `handle_keyrelease`
- movement keys + camera rotation + `Esc`

### Bonus
- dispatcher combines base + bonus handlers:
  - `main_keypress = handle_keypress + handle_keypress_bonus`
  - `main_keyrelease = handle_keyrelease + handle_keyrelease_bonus`
- adds interaction/shoot/sprint/weapon-switch
- mouse motion rotates camera (`mouse_move`)

### Input dispatch ASCII

```text
X11 Event
  |
  +--> KeyPress ------------------------------+
  |                                           |
  |                                   [MANDATORY BUILD]
  |                                           |
  |                                    handle_keypress()
  |                                    - W/A/S/D, arrows
  |                                    - ESC -> clean_exit
  |                                           |
  |                                   [BONUS BUILD]
  |                                           |
  |                                    main_keypress()
  |                                      | \
  |                                      |  +--> handle_keypress_bonus()
  |                                      |       - E, SPACE, SHIFT, 1, 2
  |                                      +----> handle_keypress()
  |
  +--> KeyRelease ---------------------------> handle_keyrelease*()
  |                                             (base + bonus in bonus build)
  |
  +--> MotionNotify (mouse) ---------------> mouse_move()
                                                -> rotate_player()
```

## 5) Parsing and validation

Pipeline:
1. Header parse (textures and floor/ceiling colors)
2. Map extraction into normalized grid
3. Player spawn detection
4. Border enclosure checks
5. Flood-fill reachability checks

Allowed map chars:
- Mandatory set: `01NSEW `
- Bonus set (compile-time): `01NSEW2DM `

Validation still enforces enclosed map constraints.

## 6) Rendering architecture

Mandatory rendering stack (`src/rendering/`):
- ray setup and DDA stepping
- wall hit detection
- texture coordinate selection
- stripe drawing
- floor/ceiling fill

Bonus rendering extensions (`bonus_src/rendering/`, `bonus_src/hud/`, `bonus_src/minimap/`, `bonus_src/sprite/`):
- UI compositing
- minimap raster
- sprite projection/sorting
- weapon + muzzle flash overlays

## 7) Bonus subsystems

### Doors
- map char: `D`
- initialized by scanning map
- runtime open/close states with cooldown
- interaction tied to player-facing tile and key `E`

### Sprites and enemies
- map char `2`: decorative/animated sprite frames
- map char `M`: monster entity with HP/state transitions
- projected in camera space and z-buffer-checked

### Weapons and HUD
- multiple guns, animation frames, muzzle flash
- shoot requests from keyboard/mouse
- HUD overlays: health, ammo, crosshair, weapon labels

### Audio
- SDL2_mixer-backed playback
- used for shooting, doors, and enemy state reactions

## 8) Directory ownership (developer mental model)

- `src/main/`: app bootstrap, MLX init, resource init, game loop
- `src/parsing/`: `.cub` parsing + validation
- `src/events/`: keyboard dispatch + movement/collision helpers
- `src/rendering/`: core raycasting renderer
- `src/utils/`: error handling, cleanup, timing helpers
- `bonus_src/*`: bonus-only gameplay/rendering/input/audio modules

## 9) Extension guidelines

- Prefer adding shared behavior in `src/` and gate bonus extras with `#ifdef BONUS` only when needed.
- Keep per-feature bonus code in `bonus_src/<feature>/` to preserve fallback build behavior.
- If you add new map symbols, update:
  1) allowed char set,
  2) validation,
  3) runtime initialization scan.
- Preserve deterministic cleanup paths (`error_exit`, `clean_exit`, resource free functions).
