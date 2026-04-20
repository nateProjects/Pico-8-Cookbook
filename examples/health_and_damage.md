# Health and Damage

Core pattern: a hit point counter, a `take_damage` function, and invincibility frames (iframes) that prevent the player being hit multiple times in one encounter.

---

## Player Setup

```lua
player = {
  x=60,  y=60,
  hp=5,  max_hp=5,
  iframes=0,    -- frames of invincibility remaining
  spr=1,
}
```

---

## Taking Damage

```lua
function take_damage(amount)
  -- do nothing during invincibility window
  if player.iframes > 0 then return end

  player.hp -= amount
  player.iframes = 45  -- ~1.5 seconds of invincibility at 30fps

  sfx(1)  -- play hurt sound

  if player.hp <= 0 then
    player.hp = 0
    on_death()
  end
end

function on_death()
  state = "gameover"  -- see game_states.md
end
```

---

## Updating Iframes Each Frame

```lua
function _update()
  -- count iframes down
  if player.iframes > 0 then player.iframes -= 1 end

  -- check contact with enemies
  for e in all(enemies) do
    if overlaps(player, e) then
      take_damage(1)
    end
  end
end

-- simple 8x8 box overlap
function overlaps(a, b)
  return abs(a.x - b.x) < 8 and abs(a.y - b.y) < 8
end
```

---

## Drawing the Player (Flicker During Iframes)

Alternate between visible and invisible every 4 frames while invincible.

```lua
function draw_player()
  -- hide sprite on alternate 4-frame windows
  if player.iframes > 0 and player.iframes % 8 < 4 then
    return
  end
  spr(player.spr, player.x, player.y)
end
```

Call `draw_player()` from `_draw()` instead of calling `spr()` directly.

---

## Healing

```lua
function heal(amount)
  player.hp = min(player.hp + amount, player.max_hp)
end
```

---

## Enemy Health

The same pattern works for enemies. Store `hp` on each enemy table and remove when depleted.

```lua
function hit_enemy(e, amount)
  e.hp -= amount
  if e.hp <= 0 then
    -- optional: drop an item, play death sound, spawn particles
    sfx(2)
    del(enemies, e)
  end
end
```

Check bullets against enemies each frame:

```lua
for b in all(bullets) do
  for e in all(enemies) do
    if overlaps(b, e) then
      hit_enemy(e, 1)
      del(bullets, b)
      break
    end
  end
end
```

---

## Knockback

Apply a velocity impulse on hit to push the player away from the source.

```lua
function take_damage_from(amount, source)
  if player.iframes > 0 then return end

  player.hp     -= amount
  player.iframes = 45

  -- push away from the damage source
  local dx = player.x - source.x
  local dy = player.y - source.y
  local d  = sqrt(dx*dx + dy*dy)
  if d > 0 then
    player.dx = (dx/d) * 4   -- knock back 4px/frame
    player.dy = (dy/d) * 4
  end

  if player.hp <= 0 then on_death() end
end
```

`player.dx` and `player.dy` then decay naturally with friction each frame (see [inertia.md](inertia.md)).

---

## Notes

- 45 iframes (~1.5s) is a common value. Shorter (20) feels snappier for fast-paced games; longer (60+) is more forgiving for beginners.
- Always clamp `hp` to 0 on death so the HUD doesn't display negative health.
- For an armour/shield system, add a second layer: `player.shield > 0` absorbs damage before `hp` is reduced.
