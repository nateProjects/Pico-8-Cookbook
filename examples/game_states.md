# Game States

A `state` variable controls which update and draw logic runs each frame. This is the foundation for title screens, game over screens, pause menus, and any screen transition.

---

## The Pattern

```lua
state = "title"  -- starts here; other values: "play", "gameover"

function _update()
  if     state == "title"    then update_title()
  elseif state == "play"     then update_play()
  elseif state == "gameover" then update_gameover()
  end
end

function _draw()
  cls()
  if     state == "title"    then draw_title()
  elseif state == "play"     then draw_play()
  elseif state == "gameover" then draw_gameover()
  end
end
```

Switching state is just an assignment: `state = "gameover"`. Put that wherever the trigger occurs (player dies, timer runs out, etc.).

---

## Full Example

```lua
-- game data
player = {}
score  = 0

function _init()
  state = "title"
end

-- ---- title ----

function update_title()
  if btnp(4) or btnp(5) then
    init_game()
    state = "play"
  end
end

function draw_title()
  print("my game",    44, 48, 7)
  print("press \142 to start", 24, 64, 6)  -- \142 = z button glyph
end

-- ---- play ----

function init_game()
  player = {x=60, y=60, hp=5, spr=1}
  score  = 0
end

function update_play()
  if btn(0) then player.x -= 2 end
  if btn(1) then player.x += 2 end

  if player.hp <= 0 then
    state = "gameover"
  end
end

function draw_play()
  map()
  spr(player.spr, player.x, player.y)
  camera()
  print("score:"..score, 2, 2, 7)
end

-- ---- game over ----

function update_gameover()
  if btnp(4) or btnp(5) then
    state = "title"
  end
end

function draw_gameover()
  print("game over",  40, 52, 8)
  print("score:"..score, 40, 64, 7)
  print("press \142",  48, 76, 6)
end
```

---

## Adding a Pause State

```lua
function update_play()
  if btnp(6) then   -- pause button (enter/p)
    state = "pause"
    return
  end
  -- ... rest of play update
end

function update_pause()
  if btnp(6) then state = "play" end
end

function draw_pause()
  -- draw the game behind the overlay
  draw_play()
  -- dim the screen
  rectfill(0, 0, 127, 127, 0)  -- semi-transparent feel via dark overlay
  print("paused", 46, 58, 7)
  print("press \134 to resume", 20, 68, 6)  -- \134 = p glyph
end
```

Add `elseif state == "pause"` to both `_update` and `_draw`.

---

## Notes

- Keep `init_game()` separate from `_init()` so you can call it on restart without rebooting the cart.
- Any state can transition to any other — the `state` variable is just a string you compare and set.
- For more complex flows (e.g. level select → cutscene → play), the same pattern scales: just add more state values and matching update/draw functions.
