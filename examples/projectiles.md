# Projectiles

Store active projectiles in a table. Each frame, move them, check for hits, and remove them when they go off-screen or collide.

---

## Basic Horizontal Bullet

```lua
player  = {x=60, y=60, facing=1}  -- facing: 1=right, -1=left
bullets = {}

function _update()
  -- move player
  if btn(0) then player.x -= 2  player.facing = -1 end
  if btn(1) then player.x += 2  player.facing =  1 end

  -- fire on button press (not held)
  if btnp(5) then
    add(bullets, {
      x  = player.x + (player.facing == 1 and 8 or -2),
      y  = player.y + 3,
      dx = player.facing * 5,
    })
  end

  -- move bullets and remove when off screen
  for b in all(bullets) do
    b.x += b.dx
    if b.x < -4 or b.x > 132 then
      del(bullets, b)
    end
  end
end

function _draw()
  cls()
  spr(1, player.x, player.y)
  for b in all(bullets) do
    rectfill(b.x, b.y+1, b.x+3, b.y+2, 7)
  end
end
```

---

## Tile Collision

Remove a bullet when it hits a solid tile (flag 0).

```lua
for b in all(bullets) do
  b.x += b.dx
  b.y += b.dy

  if b.x < 0 or b.x > 127 or b.y < 0 or b.y > 127 then
    del(bullets, b)
  elseif solid(b.x, b.y) then
    -- optional: spawn impact effect here
    del(bullets, b)
  end
end

function solid(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 0)
end
```

---

## Hitting Enemies

Check each bullet against each enemy. Remove both on contact.

```lua
enemies = {
  {x=80, y=40, hp=2},
  {x=30, y=70, hp=1},
}

function check_hits()
  for b in all(bullets) do
    for e in all(enemies) do
      if abs(b.x - e.x) < 8 and abs(b.y - e.y) < 8 then
        e.hp -= 1
        del(bullets, b)
        if e.hp <= 0 then del(enemies, e) end
        break  -- bullet is gone, stop checking enemies
      end
    end
  end
end
```

Call `check_hits()` from `_update()` after moving the bullets.

---

## Arcing Projectile (With Gravity)

Add a `dy` component and apply gravity each frame for a thrown or lobbed arc.

```lua
function throw(x, y, facing)
  add(bullets, {
    x  = x,
    y  = y,
    dx = facing * 3,
    dy = -3.5,       -- initial upward velocity
  })
end

-- in the bullet update loop:
b.dy = min(b.dy + 0.3, 5)  -- gravity
b.x  += b.dx
b.y  += b.dy
```

---

## 8-Direction Shooting

Fire in the direction the player is currently pressing.

```lua
function fire_8way()
  local dx = (btn(1) and 1 or 0) - (btn(0) and 1 or 0)
  local dy = (btn(3) and 1 or 0) - (btn(2) and 1 or 0)

  -- don't fire if no direction pressed
  if dx == 0 and dy == 0 then return end

  -- normalise diagonal speed (approx)
  local spd = 4
  if dx ~= 0 and dy ~= 0 then spd = 3 end

  add(bullets, {
    x  = player.x + 4,
    y  = player.y + 4,
    dx = dx * spd,
    dy = dy * spd,
  })
end
```

---

## Notes

- Keep a cap on the number of active bullets to avoid performance issues: `if #bullets < 8 then add(...) end`.
- `btnp(5)` fires once per press. Use `btn(5)` if you want automatic fire while held.
- `break` in the hit-check loop is important — once a bullet is deleted, continuing to reference `b` in the inner loop will cause errors.
