# Jumping and Falling

Uses velocity (`dy`) and gravity to simulate a jump arc. Ground detection reads map tile flags — mark your solid tiles with **flag 0** in the sprite editor.

---

## How It Works

- `dy` is vertical velocity. Negative = moving up, positive = moving down.
- Each frame gravity adds to `dy`, pulling the player down.
- Fall speed is capped so the player can't pass through thin tiles.
- `on_ground` is only true when a solid tile is directly underfoot, which prevents mid-air jumping.

---

## Example

```lua
-- set flag 0 on solid tiles in the sprite editor

player = {x=60, y=40, dy=0, on_ground=false}

function _update()
  -- jump: only allowed when standing on ground
  if btnp(4) and player.on_ground then
    player.dy = -4.5
  end

  -- gravity: accelerate downward each frame, capped at max fall speed
  player.dy = min(player.dy + 0.4, 4)
  player.y += player.dy

  -- landing: check two points along the bottom edge of the sprite
  player.on_ground = false
  if player.dy >= 0 and
    (solid(player.x+1, player.y+8) or solid(player.x+6, player.y+8)) then
      player.y = flr(player.y/8)*8  -- snap feet to top of tile
      player.dy = 0
      player.on_ground = true
  end

  -- head bump: stop upward movement when hitting a tile from below
  if player.dy < 0 and
    (solid(player.x+1, player.y) or solid(player.x+6, player.y)) then
      player.y = flr(player.y/8)*8 + 8  -- snap head below tile
      player.dy = 0
  end

  -- horizontal movement
  if btn(0) then player.x -= 2 end
  if btn(1) then player.x += 2 end
end

-- returns true if screen pixel (px,py) sits on a solid (flag 0) tile
function solid(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 0)
end

function _draw()
  cls()
  map()
  spr(1, player.x, player.y)
end
```

---

## One-Way Platforms (Fall-Through)

Mark platforms with **flag 1** instead of flag 0. Only block downward movement (landing), never upward. Hold DOWN to drop through them.

```lua
-- replace the landing block above with this:
player.on_ground = false

local dropping = btn(3)  -- down button held

if player.dy >= 0 and not dropping then
  -- flag 0 = solid wall (blocks both ways, handled above)
  -- flag 1 = one-way platform (only blocks landing)
  if platform(player.x+1, player.y+8) or platform(player.x+6, player.y+8) then
    player.y = flr(player.y/8)*8
    player.dy = 0
    player.on_ground = true
  end
end

-- checks flag 1 (one-way platform) on a tile
function platform(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 1)
end
```

---

## Tuning

| Variable | Effect |
|----------|--------|
| `player.dy = -4.5` | Jump strength — more negative = higher jump |
| `+ 0.4` | Gravity — larger = snappier, smaller = floatier |
| `min(..., 4)` | Max fall speed — prevents clipping through thin tiles |
