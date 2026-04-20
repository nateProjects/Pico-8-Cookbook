# Camera and Scrolling Map

`CAMERA(x, y)` shifts all drawing operations by `-x, -y`. This lets you work in **world coordinates** rather than screen coordinates — move the camera, not every object.

`CAMERA()` with no arguments resets to 0,0.

---

## Basic Pattern

```lua
function _draw()
  cls()

  -- move camera so the player appears centred on screen
  camera(player.x - 64, player.y - 64)

  -- draw world (map + entities) in world coordinates
  map()
  spr(player.spr, player.x, player.y)

  -- reset camera before drawing the HUD
  camera()
  print("hp:"..player.hp, 2, 2, 7)
end
```

The key is **resetting the camera before any HUD drawing**, otherwise health bars, scores etc. will scroll with the world.

---

## Clamping the Camera to Map Bounds

Without clamping, the camera can scroll past the edge of the map, showing empty space. `MID` clamps a value between a minimum and maximum:

```lua
-- map is 128 tiles wide, 32 tiles tall (each tile is 8px)
local map_w = 128 * 8  -- 1024px
local map_h = 32  * 8  -- 256px

local cam_x = mid(0, player.x - 64, map_w - 128)
local cam_y = mid(0, player.y - 64, map_h - 128)
camera(cam_x, cam_y)
```

Adjust `map_w` / `map_h` to match your actual map size.

---

## Drawing Only Part of the Map

`map()` with no arguments draws the entire map. You can limit what gets drawn:

`map(tile_x, tile_y, screen_x, screen_y, tiles_wide, tiles_tall)`

```lua
-- draw a 16x16 tile region starting at map tile (0,0), placed at screen (0,0)
map(0, 0, 0, 0, 16, 16)
```

When using a camera, it's usually simplest to just call `map()` and let the camera handle positioning.

---

## Full Scrolling Example

```lua
-- world coordinates
player = { x=64, y=64, spr=1 }

function _update()
  if btn(0) then player.x -= 2 end
  if btn(1) then player.x += 2 end
  if btn(2) then player.y -= 2 end
  if btn(3) then player.y += 2 end
end

function _draw()
  cls()

  -- clamp camera to map bounds (128x32 tile map)
  local cx = mid(0, player.x - 64, 128*8 - 128)
  local cy = mid(0, player.y - 64, 32*8  - 128)
  camera(cx, cy)

  map()
  spr(player.spr, player.x, player.y)

  -- hud in screen space
  camera()
  print("x:"..player.x.." y:"..player.y, 2, 2, 6)
end
```

---

## Smooth Camera Follow

The examples above snap the camera instantly to the player. For a smoother feel, lerp (linearly interpolate) the camera position toward its target each frame — it eases in and lags slightly behind fast movement.

```lua
-- initialise camera position once
cam_x = player.x - 64
cam_y = player.y - 64

function _draw()
  cls()

  -- lerp 15% toward target each frame
  cam_x += (player.x - 64 - cam_x) * 0.15
  cam_y += (player.y - 64 - cam_y) * 0.15

  -- still clamp to map bounds if needed
  cam_x = mid(0, cam_x, 128*8 - 128)
  cam_y = mid(0, cam_y, 32*8  - 128)

  camera(cam_x, cam_y)
  map()
  spr(player.spr, player.x, player.y)

  camera()
  -- hud here
end
```

The `0.15` factor controls the lag — smaller values trail further behind, larger values are closer to instant. `0.1` is dreamy, `0.25` is snappy but still smooth.

---

## Notes

- `map()` draws sprite 0 as empty by default (nothing drawn). Use `POKE(0x5F36, 0x8)` to disable this if you need sprite 0 to appear.
- The `layers` parameter of `map()` lets you draw only sprites with specific flags set — useful for layered maps (background pass, then foreground pass).
- Camera coordinates are in **pixels**, not tiles.
- `cam_x` and `cam_y` must be declared outside `_draw()` so they persist between frames — declare them at the top level alongside your other globals.
