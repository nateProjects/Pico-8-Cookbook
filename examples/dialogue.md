# Dialogue and Text Boxes

A text box at the bottom of the screen that reveals text one character at a time (typewriter effect) and advances to the next line on a button press.

---

## Data and State

```lua
-- dialogue state — nil when no dialogue is active
dlg = nil

-- internal structure (set by show_dialogue):
-- dlg.lines  : table of strings
-- dlg.index  : current line index
-- dlg.text   : full text of current line
-- dlg.pos    : characters revealed so far (float)
-- dlg.speed  : characters revealed per frame
```

---

## Core Functions

```lua
-- start a dialogue with a list of lines
function show_dialogue(lines)
  dlg = {
    lines = lines,
    index = 1,
    text  = lines[1],
    pos   = 0,
    speed = 1.5,  -- chars per frame; increase for faster reveal
  }
end

function close_dialogue()
  dlg = nil
end

function dialogue_active()
  return dlg ~= nil
end
```

---

## Update and Draw

```lua
function update_dialogue()
  if not dlg then return end

  -- reveal characters
  dlg.pos = min(dlg.pos + dlg.speed, #dlg.text)

  -- button press: finish line or advance
  if btnp(4) or btnp(5) then
    if dlg.pos < #dlg.text then
      -- skip to end of current line instantly
      dlg.pos = #dlg.text
    else
      -- move to next line, or close if finished
      dlg.index += 1
      if dlg.index > #dlg.lines then
        close_dialogue()
      else
        dlg.text = dlg.lines[dlg.index]
        dlg.pos  = 0
      end
    end
  end
end

function draw_dialogue()
  if not dlg then return end

  -- box background and border
  rectfill(4, 98, 123, 125, 0)
  rect    (4, 98, 123, 125, 7)

  -- revealed text (sub extracts first n characters)
  local visible = sub(dlg.text, 1, flr(dlg.pos))
  print(visible, 8, 103, 7)

  -- "press to continue" indicator — blinks when line is fully revealed
  if dlg.pos >= #dlg.text then
    if (t() * 3) % 1 > 0.5 then
      print("\142", 115, 117, 6)  -- z button glyph
    end
  end
end
```

Call `update_dialogue()` inside `_update()` and `draw_dialogue()` at the end of `_draw()` after `camera()`.

---

## Full Example

```lua
function _init()
  player = {x=60, y=60, spr=1}
  npc    = {x=80, y=60, spr=4}
end

function _update()
  -- block player movement during dialogue
  if not dialogue_active() then
    if btn(0) then player.x -= 2 end
    if btn(1) then player.x += 2 end

    -- start dialogue when player touches npc
    if abs(player.x - npc.x) < 12 and abs(player.y - npc.y) < 12 then
      show_dialogue({
        "hello, traveller.",
        "the dungeon lies to the north.",
        "take care — the shadows move.",
      })
    end
  end

  update_dialogue()
end

function _draw()
  cls()
  map()
  spr(npc.spr,    npc.x,    npc.y)
  spr(player.spr, player.x, player.y)
  camera()
  draw_dialogue()
end
```

---

## Speaker Name

Add a name label above the text box.

```lua
function show_dialogue(lines, speaker)
  dlg = {
    lines   = lines,
    speaker = speaker or "",  -- optional
    index   = 1,
    text    = lines[1],
    pos     = 0,
    speed   = 1.5,
  }
end

-- in draw_dialogue(), before printing the text:
if dlg.speaker ~= "" then
  rectfill(6, 92, 6 + #dlg.speaker * 4 + 4, 99, 0)
  rect    (6, 92, 6 + #dlg.speaker * 4 + 4, 99, 7)
  print(dlg.speaker, 8, 94, 10)  -- yellow name
end
```

---

## Multi-Line Text Within One Box

Split a long string across two rows manually, or let PICO-8 wrap it. Each entry in `lines` can include `\n` for a manual line break — but the simplest approach is to keep each string short enough to fit on one line (~24 characters at default font size), and use multiple entries for separate lines.

---

## Notes

- `sub(str, 1, n)` extracts the first `n` characters — this is how the typewriter reveal works.
- Blocking input during dialogue (`if not dialogue_active()`) prevents accidental movement and item pickup while reading.
- For coroutine-driven dialogue (yield until button press, no manual state machine), see [coroutines.md](coroutines.md).
