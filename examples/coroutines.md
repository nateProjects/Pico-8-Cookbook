# Coroutines

A coroutine is a function that can pause mid-execution (`yield()`) and be resumed later. This lets you write sequential, time-based logic as straight-line code — without blocking the game loop or managing a tangle of timers and flags.

---

## The Core Idea

```lua
-- create a coroutine from a function
local co = cocreate(my_function)

-- resume it each frame until it's dead
function _update()
  if costatus(co) ~= "dead" then
    coresume(co)
  end
end

-- inside the function, yield() pauses execution
-- and hands control back to _update
function my_function()
  print("step one")
  for i = 1, 60 do yield() end  -- wait 2 seconds
  print("step two")
  for i = 1, 30 do yield() end  -- wait 1 second
  print("done")
end
```

Each call to `coresume` runs the function until the next `yield()`, then returns. The function picks up exactly where it left off on the next resume.

---

## Cutscene Sequence

Without coroutines this would require multiple state variables and frame counters. With a coroutine it reads like a script.

```lua
cutscene = nil
message  = ""

function start_cutscene()
  cutscene = cocreate(run_cutscene)
end

function run_cutscene()
  -- pan camera to a location
  message = "the castle gates open..."
  for i = 1, 90 do yield() end   -- hold for 3 seconds

  message = "a shadow moves inside."
  for i = 1, 60 do yield() end

  message = ""
  -- hand off to gameplay
  state = "play"
end

function _update()
  if cutscene and costatus(cutscene) ~= "dead" then
    coresume(cutscene)
  end
end

function _draw()
  cls()
  map()
  -- draw letterbox bars for cinematic feel
  if cutscene then
    rectfill(0, 0,   127, 20,  0)
    rectfill(0, 107, 127, 127, 0)
    print(message, 4, 110, 7)
  end
end
```

---

## Helper: Wait N Frames

Extract the wait loop into a helper to keep sequences readable.

```lua
function wait(frames)
  for i = 1, frames do yield() end
end

-- usage inside a coroutine:
function run_intro()
  message = "level 1"  wait(60)
  message = "get ready" wait(45)
  message = ""
  state   = "play"
end
```

---

## Dialogue Sequence

Button-gated coroutine — each line waits for a button press rather than a fixed duration. See [dialogue.md](dialogue.md) for the full text-box version; this shows the coroutine side.

```lua
local seq = nil

function start_dialogue(lines)
  seq = cocreate(function()
    for line in all(lines) do
      current_line = line
      repeat yield() until btnp(4) or btnp(5)
    end
    current_line = nil
    seq = nil
  end)
end

function _update()
  if seq then coresume(seq) end
end
```

---

## Multiple Simultaneous Coroutines

Store them in a table and tick all of them each frame. Remove finished ones.

```lua
routines = {}

function run(fn)
  add(routines, cocreate(fn))
end

function _update()
  for co in all(routines) do
    if costatus(co) ~= "dead" then
      coresume(co)
    else
      del(routines, co)
    end
  end
end

-- start independent sequences:
run(function()
  -- enemy entrance animation
  for i = 1, 30 do yield() end
  spawn_boss()
end)

run(function()
  -- screen flash
  flash = true
  wait(8)
  flash = false
end)
```

---

## Notes

- `coresume(co)` returns `false, error_message` if the coroutine errors. Wrap with `assert(coresume(co))` during development to surface errors immediately rather than silently dying.
- A coroutine that reaches the end of its function has `costatus == "dead"` and cannot be resumed again. Create a new one if you need to replay a sequence.
- `yield()` can only be called from inside a coroutine — calling it in a regular function will error.
