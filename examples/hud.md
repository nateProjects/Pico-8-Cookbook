# HUD — Game Stats and Health Bar

The HUD (heads-up display) shows score, lives, timers, and health. Always draw it **after** calling `camera()` with no arguments to reset to screen space, otherwise it scrolls with the world.

```lua
function _draw()
  cls()

  -- draw world (in world/camera space)
  camera(cam_x, cam_y)
  map()
  spr(player.spr, player.x, player.y)

  -- reset to screen space before drawing HUD
  camera()
  draw_hud()
end
```

---

## Score and Lives

```lua
score = 0
lives = 3

function draw_hud()
  -- score: top-left
  print("score:"..score, 2, 2, 7)

  -- lives: draw icon sprites in a row
  for i = 1, lives do
    spr(16, 90 + (i - 1) * 10, 1)  -- sprite 16 = heart icon
  end
end
```

---

## Countdown Timer

```lua
timer = 30 * 60  -- 60 seconds

function draw_hud()
  local secs  = ceil(timer / 30)
  local col   = secs <= 10 and 8 or 7  -- red when low
  print("time:"..secs, 52, 2, col)
end
```

---

## Health Bar

A filled rectangle that shrinks as HP drops, with a background track showing the maximum.

```lua
-- draw_health_bar(x, y, w, h, hp, max_hp)
-- x,y = top-left corner  w,h = total bar size in pixels

function draw_health_bar(x, y, w, h, hp, max_hp)
  -- background track (empty bar)
  rectfill(x, y, x + w - 1, y + h - 1, 5)

  -- filled portion (scales with hp/max_hp)
  local fill = flr((w - 1) * (hp / max_hp))
  if fill > 0 then
    rectfill(x, y, x + fill, y + h - 1, 8)  -- red fill
  end

  -- border
  rect(x, y, x + w - 1, y + h - 1, 7)
end
```

**Usage:**
```lua
draw_health_bar(2, 120, 60, 6, player.hp, player.max_hp)
```

Colour ideas: green `(11)` for friendly, red `(8)` for enemy/danger, yellow `(10)` for shield/armour.

---

## Full HUD Example

```lua
function draw_hud()
  -- health bar: bottom-left
  draw_health_bar(2, 120, 60, 6, player.hp, player.max_hp)
  print("hp", 2, 110, 6)

  -- score: top-left
  print(score, 2, 2, 7)

  -- lives (heart icons): top-right
  for i = 1, lives do
    spr(16, 128 - (i * 10), 1)
  end

  -- wave / level: top-centre
  print("wave "..wave, 48, 2, 6)

  -- countdown timer: turns red below 10 seconds
  local secs = ceil(timer / 30)
  print(secs, 60, 120, secs <= 10 and 8 or 7)
end
```

---

## Enemy Health Bar (World Space)

For a boss or mini-boss, draw the bar above the enemy in world coordinates — before resetting the camera.

```lua
function draw_boss_bar(boss)
  -- draw in world space (camera already set)
  local bx = boss.x - 8
  local by = boss.y - 6
  draw_health_bar(bx, by, 24, 3, boss.hp, boss.max_hp)
end
```

---

## Notes

- `ceil(timer / 30)` rounds up so the display never shows 0 while there's still time left — it jumps from 1 to 0 exactly when the timer fires.
- For a two-colour bar (e.g. yellow inner + red outer for damage flash), draw the flash colour first, then the current HP fill on top.
- Keep HUD elements at least 2px from the screen edge so they don't get cut off on hardware or exported builds.
