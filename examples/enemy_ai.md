# Basic Enemy AI

Enemies have a `state` that controls their behaviour. The two most common states are **patrol** (move back and forth on a path) and **chase** (move toward the player). A detection range switches between them.

---

## Patrol + Chase

```lua
player = {x=64, y=64}

enemy = {
  x=40,  y=50,
  state="patrol",
  dir=1,           -- 1=right, -1=left
  patrol_x1=20,    -- left boundary of patrol path
  patrol_x2=80,    -- right boundary
  speed=1,
  detect_range=40, -- how close the player must be to trigger chase
  spr=8,
}

function update_enemy(e)
  if e.state == "patrol" then
    -- walk back and forth between patrol bounds
    e.x += e.dir * e.speed
    if e.x >= e.patrol_x2 then e.dir = -1 end
    if e.x <= e.patrol_x1 then e.dir =  1 end

    -- switch to chase when player enters range
    if dist(e, player) < e.detect_range then
      e.state = "chase"
    end

  elseif e.state == "chase" then
    -- move directly toward the player
    local dx = player.x - e.x
    local dy = player.y - e.y
    local d  = sqrt(dx*dx + dy*dy)
    if d > 0 then
      e.x += (dx / d) * e.speed * 1.5
      e.y += (dy / d) * e.speed * 1.5
    end

    -- give up if player escapes twice the detection range
    if dist(e, player) > e.detect_range * 2 then
      e.state = "patrol"
    end
  end
end

function dist(a, b)
  local dx = a.x - b.x
  local dy = a.y - b.y
  return sqrt(dx*dx + dy*dy)
end

function _update()
  -- player movement
  if btn(0) then player.x -= 2 end
  if btn(1) then player.x += 2 end
  if btn(2) then player.y -= 2 end
  if btn(3) then player.y += 2 end

  update_enemy(enemy)
end

function _draw()
  cls()
  spr(1, player.x, player.y)
  spr(enemy.spr, enemy.x, enemy.y)

  -- debug: show detection range
  -- circ(enemy.x+4, enemy.y+4, enemy.detect_range, 5)
end
```

---

## Multiple Enemies

The same `update_enemy` function works for a whole table of enemies.

```lua
enemies = {
  {x=20, y=50, state="patrol", dir=1, patrol_x1=10, patrol_x2=60,
   speed=1, detect_range=40, spr=8},
  {x=100, y=30, state="patrol", dir=-1, patrol_x1=80, patrol_x2=120,
   speed=1.5, detect_range=32, spr=9},
}

function _update()
  -- player movement (as above)
  for e in all(enemies) do
    update_enemy(e)
  end
end

function _draw()
  cls()
  spr(1, player.x, player.y)
  for e in all(enemies) do
    spr(e.spr, e.x, e.y)
  end
end
```

---

## Adding a Third State: Attack

When the enemy is close enough, stop moving and attack.

```lua
function update_enemy(e)
  if e.state == "patrol" then
    -- (as above)
    if dist(e, player) < e.detect_range then e.state = "chase" end

  elseif e.state == "chase" then
    -- move toward player
    local dx = player.x - e.x
    local dy = player.y - e.y
    local d  = sqrt(dx*dx + dy*dy)
    if d > 0 then
      e.x += (dx / d) * e.speed
      e.y += (dy / d) * e.speed
    end

    if dist(e, player) < 10 then
      e.state   = "attack"
      e.atk_cd  = 0
    elseif dist(e, player) > e.detect_range * 2 then
      e.state = "patrol"
    end

  elseif e.state == "attack" then
    -- stand still and hit the player on cooldown
    if e.atk_cd > 0 then
      e.atk_cd -= 1
    else
      take_damage(1)     -- see health_and_damage.md
      e.atk_cd = 30      -- attack every 30 frames
    end

    -- return to chase if player moves away
    if dist(e, player) > 12 then
      e.state = "chase"
    end
  end
end
```

---

## Notes

- The `(dx/d) * speed` movement normalises the direction vector so diagonal movement isn't faster than axis-aligned.
- For tile-based movement (grid RPG style), round positions to the tile grid and move one tile at a time rather than floating point.
- For larger maps, consider only running `update_enemy` on enemies within a certain distance of the player to save CPU.
