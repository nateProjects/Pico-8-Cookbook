# Movement Inertia

Instead of moving at a fixed speed, the player accelerates up to a maximum and decelerates when input stops. This gives a sliding, momentum-based feel.

---

## How It Works

- Pressing a direction adds to velocity (`dx`) up to a cap.
- Releasing input multiplies `dx` by a friction value (less than 1) each frame, causing it to bleed off.
- Very small values are snapped to 0 to prevent endless micro-sliding.

---

## Example

```lua
player = {x=60, y=60, dx=0, dy=0}

local accel     = 0.5   -- speed gained per frame when pressing
local max_speed = 3     -- top speed
local friction  = 0.75  -- speed multiplied each frame when not pressing (0–1)
local stop_threshold = 0.1  -- snap to 0 below this speed

function _update()
  local moving_x = false
  local moving_y = false

  if btn(0) then player.dx -= accel  moving_x = true end
  if btn(1) then player.dx += accel  moving_x = true end
  if btn(2) then player.dy -= accel  moving_y = true end
  if btn(3) then player.dy += accel  moving_y = true end

  -- clamp to max speed
  player.dx = mid(-max_speed, player.dx, max_speed)
  player.dy = mid(-max_speed, player.dy, max_speed)

  -- apply friction when no input
  if not moving_x then
    player.dx *= friction
    if abs(player.dx) < stop_threshold then player.dx = 0 end
  end
  if not moving_y then
    player.dy *= friction
    if abs(player.dy) < stop_threshold then player.dy = 0 end
  end

  player.x += player.dx
  player.y += player.dy
end

function _draw()
  cls()
  spr(1, player.x, player.y)
end
```

---

## Tuning the Feel

| Setting | Low value | High value |
|---------|-----------|------------|
| `accel` | Slow to get up to speed | Snappy, near-instant |
| `max_speed` | Slow character | Fast character |
| `friction` (0–1) | Slides a long way | Stops almost instantly |

**Tight / responsive** (platformer):
```lua
local accel    = 1.2
local max_speed = 3
local friction  = 0.6
```

**Loose / floaty** (ice level, space):
```lua
local accel    = 0.3
local max_speed = 4
local friction  = 0.92
```

---

## Inertia With Tile Collision

Apply position separately per axis, revert on collision — same approach as [wall_collision_function.src](wall_collision_function.src).

```lua
function _update()
  -- (acceleration and friction as above)

  -- move x, revert if solid
  local prev_x = player.x
  player.x += player.dx
  if solid(player.x, player.y) or solid(player.x+7, player.y) or
     solid(player.x, player.y+7) or solid(player.x+7, player.y+7) then
    player.x  = prev_x
    player.dx = 0
  end

  -- move y, revert if solid
  local prev_y = player.y
  player.y += player.dy
  if solid(player.x, player.y) or solid(player.x+7, player.y) or
     solid(player.x, player.y+7) or solid(player.x+7, player.y+7) then
    player.y  = prev_y
    player.dy = 0
  end
end

function solid(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 0)
end
```

---

## Notes

- Checking both axes independently means the player slides along walls instead of stopping dead when hitting at an angle.
- For a platformer, replace the vertical friction block with gravity (see [jumping.md](jumping.md)) — you usually don't want friction on `dy`.
