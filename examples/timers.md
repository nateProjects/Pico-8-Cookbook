# Timers and Cooldowns

PICO-8 runs at 30fps by default. Timers are just integers that count down (or up) each frame.

**1 second = 30 frames.** Use this to convert between human-readable durations and frame counts.

---

## Countdown Timer

Decrements each frame, triggers something when it reaches zero.

```lua
timer = 30 * 60  -- 60 seconds (at 30fps)

function _update()
  if timer > 0 then
    timer -= 1
  end

  if timer == 0 then
    -- time's up — trigger game over, next wave, etc.
    state = "gameover"
  end
end

function _draw()
  -- display as seconds remaining
  local secs = ceil(timer / 30)
  print("time: "..secs, 2, 2, secs <= 10 and 8 or 7)  -- red when low
end
```

---

## Cooldown (Rate Limiting)

Prevents an action from being repeated until enough frames have passed.

```lua
attack_cd = 0

function _update()
  if attack_cd > 0 then attack_cd -= 1 end

  if btnp(5) and attack_cd == 0 then
    fire()
    attack_cd = 15  -- can't fire again for 15 frames (~0.5s)
  end
end
```

Any action works the same way: dashing, using an item, taking damage (see [health_and_damage.md](health_and_damage.md)).

---

## Repeating Timer (Spawn / Event Interval)

Resets itself each time it fires.

```lua
spawn_timer = 60  -- first spawn after 2 seconds

function _update()
  spawn_timer -= 1
  if spawn_timer <= 0 then
    spawn_enemy()
    spawn_timer = 90  -- repeat every 3 seconds
  end
end
```

To make the interval shrink over time (increasing difficulty):

```lua
spawn_interval = 90

function on_wave_complete()
  spawn_interval = max(20, spawn_interval - 10)  -- faster each wave, minimum 20
end
```

---

## Delay (Do Something Once, After N Frames)

```lua
delay = 45  -- trigger after 1.5 seconds
delay_done = false

function _update()
  if not delay_done then
    delay -= 1
    if delay <= 0 then
      delay_done = true
      trigger_event()
    end
  end
end
```

---

## Multiple Independent Timers

Store timers on the objects that own them rather than as globals.

```lua
player = {x=60, y=60, attack_cd=0, dash_cd=0}

function _update()
  if player.attack_cd > 0 then player.attack_cd -= 1 end
  if player.dash_cd   > 0 then player.dash_cd   -= 1 end

  if btnp(5) and player.attack_cd == 0 then
    fire()
    player.attack_cd = 12
  end

  if btnp(4) and player.dash_cd == 0 then
    dash()
    player.dash_cd = 45
  end
end
```

---

## Reference

| Duration | Frames (30fps) |
|----------|---------------|
| 0.25s    | 8             |
| 0.5s     | 15            |
| 1s       | 30            |
| 2s       | 60            |
| 5s       | 150           |
| 1 min    | 1800          |
