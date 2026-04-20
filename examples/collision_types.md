# Collision Types

Three reusable collision functions covering the most common scenarios.

---

## 1. Pixel Colour Collision

Check whether a specific screen pixel has been drawn with a particular colour. Useful for games where collision is encoded in the image itself rather than map data.

```lua
-- returns true if the pixel at (px, py) on screen matches col
function pixel_is_color(px, py, col)
  return pget(px, py) == col
end
```

**Usage:**
```lua
-- has the player's leading edge touched anything red (colour 8)?
if pixel_is_color(player.x+8, player.y+4, 8) then
  -- hit a red object
end
```

*Note: `pget` reads the draw buffer, so this only works reliably after the relevant pixels have been drawn in `_draw`. It's best suited for pixel-art games where the map is painted rather than tile-based.*

---

## 2. Map Tile Flag Collision

Check whether a map tile at a given tile coordinate has a specific flag set. This is the most common collision method for tile-based platformers and top-down games.

Takes **tile coordinates** (not pixel coordinates) — divide pixel positions by 8.

```lua
-- returns true if the tile at (tx, ty) has flag f set
function tile_has_flag(tx, ty, f)
  return fget(mget(tx, ty), f)
end
```

**Usage:**
```lua
-- is the tile below the player solid (flag 0)?
local tx = flr(player.x/8)
local ty = flr((player.y+8)/8)  -- one row below feet
if tile_has_flag(tx, ty, 0) then
  -- standing on solid ground
end
```

In practice this is often wrapped in a helper that handles the pixel-to-tile conversion:

```lua
function solid(px, py)
  return tile_has_flag(flr(px/8), flr(py/8), 0)
end
```

See [jumping.md](jumping.md) for a full platformer example using this pattern.

---

## 3. Point–Point Overlap

Check whether two objects share the same grid position. Useful for grid-based games — turn-based RPGs, puzzle games, top-down dungeon crawlers — where everything snaps to a tile.

```lua
-- returns true if objects a and b are at the same position
function points_overlap(a, b)
  return a.x == b.x and a.y == b.y
end
```

**Usage:**
```lua
player = {x=3, y=4}
chest  = {x=3, y=4}

if points_overlap(player, chest) then
  -- player is standing on the chest
end
```

For pixel-based movement, prefer the AABB box collision in [draw_collision.src](draw_collision.src) instead — exact point matching is unreliable when positions change by more than 1 pixel per frame.

---

## Comparison

| Function | Coordinates | Best for |
|----------|-------------|----------|
| `pixel_is_color` | Screen pixels | Pixel-art / painted collision |
| `tile_has_flag` | Tile grid | Tile maps, platformers |
| `points_overlap` | Any (grid) | Turn-based, grid-snapped games |
| `collides` (see [draw_collision.src](draw_collision.src)) | Pixel rect | Freely-moving objects, AABB |
