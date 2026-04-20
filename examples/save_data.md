# Save Data — CARTDATA, DGET, DSET

PICO-8 provides 64 number slots (256 bytes) of persistent storage per cartridge. Data survives the cart being unloaded or PICO-8 shutting down.

---

## Setup

Call `cartdata(id)` **once**, at startup. It opens a named storage slot and maps it to memory.

```lua
function _init()
  cartdata("your_game_name_v1")
end
```

`id` rules:
- Up to 64 characters, lowercase only: `a–z`, `0–9`, `_`
- Should be unique — other carts sharing the same id will read each other's data
- Include your game name and a version if your save format changes

`cartdata()` returns `true` if existing data was found, `false` if it's a fresh slot.

---

## Reading and Writing

`dset(index, value)` — write a number to slot `index` (0–63)  
`dget(index)` — read the number at slot `index` (returns 0 if never written)

```lua
-- save the player's score
dset(0, score)

-- load it back
score = dget(0)
```

Data is saved automatically — no explicit flush needed.

---

## Planning Your Slot Layout

Document which slot holds what. A comment block at the top of your code works well:

```lua
-- save data layout:
-- slot 0: high score
-- slot 1: last level reached
-- slot 2: sfx volume (0–8)
-- slot 3: music on/off (0=off, 1=on)
```

---

## Full Example

```lua
-- save data layout
-- 0: high score
-- 1: level unlocked
-- 2: music enabled (0 or 1)

local hi_score = 0
local level_unlocked = 1
local music_on = true

function _init()
  cartdata("mygame_saves_v1")
  load_progress()
end

function load_progress()
  hi_score        = dget(0)
  level_unlocked  = max(1, dget(1))  -- at least level 1
  music_on        = dget(2) == 1
end

function save_progress()
  dset(0, max(hi_score, score))       -- keep the highest score ever
  dset(1, level_unlocked)
  dset(2, music_on and 1 or 0)        -- booleans stored as 0/1
end
```

---

## Storing Booleans and Small Integers

PICO-8 numbers are fixed-point (not integers), so store booleans as `0` / `1`:

```lua
dset(2, music_on and 1 or 0)
music_on = dget(2) == 1
```

For small integers (e.g. lives 0–9), just store them directly. For larger values you can pack multiple small values into one slot using bitwise ops, but usually there's no need with 64 slots available.

---

## Notes

- `cartdata()` can only be called **once** per run. Calling it again has no effect.
- The 64 slots are shared across all versions of a cart with the same id. If you change your save format in an update, bump the version in the id (e.g. `mygame_v2`) to avoid reading stale data.
- You can also read/write save memory directly via `PEEK`/`POKE` at `0x5e00–0x5eff` once `cartdata()` has been called.
