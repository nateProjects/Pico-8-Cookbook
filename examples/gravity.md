# Gravity

Gravity is just a small value added to vertical velocity (`dy`) every frame. The velocity accumulates until something stops it (a floor, a cap, a bounce).

This pattern applies to anything that falls: the player, enemies, projectiles, debris, dropped items.

---

## The Core Pattern

```lua
local gravity = 0.4
local max_fall = 4

-- each frame:
obj.dy = min(obj.dy + gravity, max_fall)
obj.y  += obj.dy
```

`min(..., max_fall)` caps the speed so fast-moving objects don't tunnel through thin tiles.

---

## Standalone Falling Object

```lua
rock = {x=64, y=0, dy=0}

function _update()
  rock.dy = min(rock.dy + 0.4, 5)
  rock.y  += rock.dy

  -- simple floor at y=120
  if rock.y >= 120 then
    rock.y  = 120
    rock.dy = 0
  end
end

function _draw()
  cls()
  spr(2, rock.x, rock.y)
end
```

---

## Arcing Projectile

Gravity on a projectile gives it a realistic arc. Fire with an upward component (`dy < 0`) and let gravity pull it down.

```lua
-- fire at 45 degrees upward to the right
arrow = {x=10, y=60, dx=3, dy=-3}

function _update()
  arrow.dy = min(arrow.dy + 0.3, 4)
  arrow.x  += arrow.dx
  arrow.y  += arrow.dy
end

function _draw()
  cls()
  -- rotate sprite to match direction of travel
  local flip_y = arrow.dy > 0
  spr(5, arrow.x, arrow.y, 1, 1, false, flip_y)
end
```

---

## Multiple Falling Objects

```lua
debris = {}

function spawn_debris(x, y)
  add(debris, {
    x  = x,
    y  = y,
    dx = rnd(4) - 2,   -- random horizontal spread
    dy = rnd(-3) - 1,  -- initial upward burst
  })
end

function _update()
  for d in all(debris) do
    d.dy = min(d.dy + 0.35, 5)
    d.x  += d.dx
    d.y  += d.dy

    -- remove when off the bottom of the screen
    if d.y > 136 then del(debris, d) end
  end
end

function _draw()
  cls()
  for d in all(debris) do
    pset(d.x, d.y, 10)
  end
end
```

---

## Tuning

| Value | Effect |
|-------|--------|
| Gravity `0.2` | Floaty, moon-like |
| Gravity `0.4` | Natural platformer feel |
| Gravity `0.8`+ | Heavy, snappy |
| `max_fall = 4` | Safe for 8px tiles |
| `max_fall = 8`+ | Needs thicker collision geometry |

See [jumping.md](jumping.md) for gravity combined with player input and tile collision.  
See [bouncing.md](bouncing.md) for gravity combined with surface bouncing.
