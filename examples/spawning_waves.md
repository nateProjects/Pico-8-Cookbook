# Spawning Waves

Two common patterns: **timed spawning** (one enemy every N frames) and **wave clearing** (spawn a group, wait until all are dead, then spawn the next group).

---

## Timed Spawning

Spawn enemies on a regular interval. Reduce the interval over time to increase difficulty.

```lua
enemies       = {}
spawn_timer   = 60   -- frames until first spawn
spawn_interval = 90  -- frames between spawns

function _update()
  spawn_timer -= 1
  if spawn_timer <= 0 then
    spawn_enemy()
    spawn_timer = spawn_interval
  end

  for e in all(enemies) do
    update_enemy(e)
  end
end

function spawn_enemy()
  -- spawn from a random screen edge
  local x, y
  if rnd(2) < 1 then
    x = rnd(128)
    y = rnd(2) < 1 and -8 or 136   -- top or bottom
  else
    x = rnd(2) < 1 and -8 or 136   -- left or right
    y = rnd(128)
  end

  add(enemies, {x=x, y=y, hp=1, speed=0.8, spr=8})
end
```

---

## Wave-Based Spawning

Spawn a batch, wait until the screen is cleared, then advance to the next wave.

```lua
enemies   = {}
wave      = 0
wave_size = 3    -- enemies in first wave

function _init()
  start_wave()
end

function start_wave()
  wave      += 1
  wave_size += wave - 1  -- one more enemy each wave

  for i = 1, wave_size do
    spawn_enemy()
  end
end

function _update()
  for e in all(enemies) do
    update_enemy(e)
  end

  -- when all enemies are dead, start the next wave
  if #enemies == 0 then
    start_wave()
  end
end
```

---

## Escalating Difficulty

Adjust enemy stats based on the current wave number.

```lua
function spawn_enemy()
  local x, y = random_edge_position()

  add(enemies, {
    x     = x,
    y     = y,
    spr   = 8,
    hp    = 1 + flr(wave / 3),          -- more hp every 3 waves
    speed = 0.5 + wave * 0.08,          -- faster each wave
    score = 10 * wave,                  -- worth more points
  })
end
```

---

## Spawning From Fixed Locations

Spawn from specific map positions (doorways, spawn points) rather than random edges.

```lua
-- mark spawn tiles with flag 2 in the sprite editor
-- collect all spawn tile positions at startup

spawn_points = {}

function find_spawn_points()
  for tx = 0, 127 do
    for ty = 0, 31 do
      if fget(mget(tx, ty), 2) then
        add(spawn_points, {x=tx*8, y=ty*8})
      end
    end
  end
end

function spawn_enemy()
  if #spawn_points == 0 then return end
  -- pick a random spawn point
  local sp = spawn_points[flr(rnd(#spawn_points)) + 1]
  add(enemies, {x=sp.x, y=sp.y, hp=1, speed=1, spr=8})
end
```

Call `find_spawn_points()` from `_init()` once.

---

## Staggered Spawning (Not All At Once)

Spawn wave enemies one at a time with a short delay between each.

```lua
to_spawn    = 0   -- enemies still queued for this wave
stagger_cd  = 0   -- frames between each spawn

function start_wave()
  wave     += 1
  to_spawn  = 3 + wave
end

function _update()
  -- stagger spawns
  if to_spawn > 0 then
    stagger_cd -= 1
    if stagger_cd <= 0 then
      spawn_enemy()
      to_spawn   -= 1
      stagger_cd  = 20  -- 20 frames between each enemy
    end
  end

  -- advance wave when all spawned and all dead
  if to_spawn == 0 and #enemies == 0 then
    start_wave()
  end

  for e in all(enemies) do update_enemy(e) end
end
```

---

## Notes

- `random_edge_position()` used above is a helper — use the edge-spawn code from the timed spawning example or inline it.
- Cap total enemies on screen with `if #enemies < max_enemies then spawn_enemy() end` to keep performance stable.
- Give the player a brief breather between waves — a 2-second pause before `start_wave()` gives them a moment to breathe and see the wave number.
