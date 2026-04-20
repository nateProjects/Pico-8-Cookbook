# Bouncing

When an object hits a surface, flip the sign of the relevant velocity component. Hitting a floor/ceiling flips `dy`; hitting a wall flips `dx`.

---

## Ball Bouncing Off Screen Edges

```lua
ball = {x=64, y=30, dx=2.5, dy=1.5, r=4}

function _update()
  ball.x += ball.dx
  ball.y += ball.dy

  -- left / right walls
  if ball.x - ball.r < 0 then
    ball.x  = ball.r
    ball.dx = -ball.dx
  end
  if ball.x + ball.r > 127 then
    ball.x  = 127 - ball.r
    ball.dx = -ball.dx
  end

  -- ceiling / floor
  if ball.y - ball.r < 0 then
    ball.y  = ball.r
    ball.dy = -ball.dy
  end
  if ball.y + ball.r > 127 then
    ball.y  = 127 - ball.r
    ball.dy = -ball.dy
  end
end

function _draw()
  cls()
  circfill(ball.x, ball.y, ball.r, 7)
end
```

Clamping the position on impact (`ball.x = ball.r`) stops the ball from tunnelling into or through the wall before the flip takes effect.

---

## Energy Loss (Damping)

Multiply the flipped velocity by a value less than 1 to lose energy on each bounce. 1.0 = perfectly elastic, 0.0 = no bounce at all.

```lua
-- bounce off floor with 20% energy loss
if ball.y + ball.r > 127 then
  ball.y  = 127 - ball.r
  ball.dy = -ball.dy * 0.8
  ball.dx =  ball.dx * 0.95  -- slight horizontal friction too
end
```

---

## Dropped Object (Falls and Bounces)

An object dropped straight down with no horizontal movement — `dx=0`. Gravity pulls it down, it bounces off the floor and loses energy until it settles.

```lua
ball = {x=64, y=10, dx=0, dy=0, r=3}

function _update()
  -- gravity
  ball.dy = min(ball.dy + 0.4, 6)

  ball.x += ball.dx
  ball.y += ball.dy

  -- floor bounce with energy loss
  if ball.y + ball.r > 120 then
    ball.y  = 120 - ball.r
    ball.dy = -ball.dy * 0.75  -- lose 25% energy each bounce

    -- small random horizontal scatter on each bounce (optional)
    -- ball.dx = rnd(1) - 0.5
  end

  -- stop micro-bouncing when nearly still
  if abs(ball.dy) < 0.3 and ball.y + ball.r >= 119 then
    ball.dy = 0
    ball.y  = 120 - ball.r
  end
end

function _draw()
  cls()
  line(0, 120, 127, 120, 5)  -- floor
  circfill(ball.x, ball.y, ball.r, 10)
end
```

---

## Thrown Object (Arc + Wall Bounces)

Combine gravity with an initial horizontal velocity for a thrown arc that bounces off floor and walls.

```lua
ball = {x=20, y=20, dx=2, dy=0, r=3}

function _update()
  ball.dy = min(ball.dy + 0.4, 6)

  ball.x += ball.dx
  ball.y += ball.dy

  -- floor bounce
  if ball.y + ball.r > 120 then
    ball.y  = 120 - ball.r
    ball.dy = -ball.dy * 0.75
    ball.dx =  ball.dx * 0.9   -- floor friction slows horizontal too
  end

  -- left / right walls
  if ball.x - ball.r < 0 or ball.x + ball.r > 127 then
    ball.dx = -ball.dx
    ball.x  = mid(ball.r, ball.x, 127 - ball.r)
  end

  -- stop micro-bouncing when nearly still
  if abs(ball.dy) < 0.3 and ball.y + ball.r >= 119 then
    ball.dy = 0
    ball.y  = 120 - ball.r
  end
end

function _draw()
  cls()
  line(0, 120, 127, 120, 5)
  circfill(ball.x, ball.y, ball.r, 9)
end
```

---

## Bouncing Off a Box Barrier

For dynamically placed barriers (not map tiles), check overlap between the ball and a rectangle and determine which side was hit by comparing how far the ball has penetrated each edge.

```lua
ball    = {x=30, y=20, dx=1.5, dy=0, r=4}
barrier = {x=40, y=60, w=48, h=12}  -- a box anywhere on screen

function _update()
  ball.dy = min(ball.dy + 0.4, 6)
  ball.x  += ball.dx
  ball.y  += ball.dy

  -- check ball centre against expanded barrier (barrier grown by ball radius)
  local bx1 = barrier.x - ball.r
  local bx2 = barrier.x + barrier.w + ball.r
  local by1 = barrier.y - ball.r
  local by2 = barrier.y + barrier.h + ball.r

  if ball.x > bx1 and ball.x < bx2 and
     ball.y > by1 and ball.y < by2 then

    -- find how far into each edge the ball has gone
    local overlap_left   = ball.x - bx1
    local overlap_right  = bx2 - ball.x
    local overlap_top    = ball.y - by1
    local overlap_bottom = by2 - ball.y

    -- bounce off the closest edge
    local min_h = min(overlap_left, overlap_right)
    local min_v = min(overlap_top,  overlap_bottom)

    if min_h < min_v then
      -- hit a side wall
      ball.dx = -ball.dx * 0.85
      if overlap_left < overlap_right then
        ball.x = bx1  -- push out left
      else
        ball.x = bx2  -- push out right
      end
    else
      -- hit top or bottom
      ball.dy = -ball.dy * 0.8
      if overlap_top < overlap_bottom then
        ball.y = by1  -- push out top
      else
        ball.y = by2  -- push out bottom
      end
    end
  end

  -- screen floor
  if ball.y + ball.r > 127 then
    ball.y  = 127 - ball.r
    ball.dy = -ball.dy * 0.75
  end
end

function _draw()
  cls()
  rectfill(barrier.x, barrier.y,
           barrier.x + barrier.w, barrier.y + barrier.h, 5)
  circfill(ball.x, ball.y, ball.r, 9)
end
```

Multiple barriers work the same way — loop through a table of them and run the same check for each.

---

## Bouncing Off Map Tiles

Check the tile in the direction of travel. Flip the matching velocity component on hit.

```lua
-- flag 0 = solid tile
function _update()
  ball.dy = min(ball.dy + 0.3, 5)
  ball.x  += ball.dx
  ball.y  += ball.dy

  -- vertical bounce
  local check_y = ball.dy > 0 and ball.y + ball.r or ball.y - ball.r
  if solid(ball.x, check_y) then
    ball.dy = -ball.dy * 0.8
    ball.y  -= ball.dy  -- push back out of tile
  end

  -- horizontal bounce
  local check_x = ball.dx > 0 and ball.x + ball.r or ball.x - ball.r
  if solid(check_x, ball.y) then
    ball.dx = -ball.dx * 0.9
    ball.x  -= ball.dx
  end
end

function solid(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 0)
end
```

---

## Notes

- Always clamp position back in-bounds after flipping velocity — otherwise the object can get trapped inside the surface.
- The "stop micro-bouncing" check (`abs(ball.dy) < 0.3`) prevents the ball rattling forever with imperceptibly small bounces.
- For angular bouncing off slopes, you need to reflect the velocity vector across the surface normal — not covered here.
