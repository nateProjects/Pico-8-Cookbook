# Pico-8 Sprite Flags

Sprite flags are 8 per-sprite boolean values (true/false), indexed **0–7**. They have no built-in meaning — you decide what each one represents (e.g. flag 0 = solid, flag 1 = hazard).

They can be set in the sprite editor (the coloured dots), or in code.

## FSET — Set a flag

`fset(sprite_num, flag_index, value)`

* `flag_index` — which flag to set: **0 to 7**
* `value` — `true` or `false`

```lua
fset(1, 0, true)   -- set flag 0 on sprite 1 (e.g. mark as solid)
fset(2, 1, true)   -- set flag 1 on sprite 2 (e.g. mark as hazard)
fset(1, 0, false)  -- clear flag 0 on sprite 1
```

To set all 8 flags at once using a bitfield:

```lua
fset(1, 0b00000011)  -- set flags 0 and 1, clear the rest
fset(2, 1 | 2 | 8)   -- set flags 0, 1 and 3 (bit values 1+2+8=11)
```

## FGET — Read a flag

`fget(sprite_num, flag_index)` → returns `true` or `false`

```lua
if fget(tile, 0) then
  -- tile has flag 0 set (e.g. solid wall)
end
```

To read all 8 flags at once as a bitfield:

`fget(sprite_num)` → returns a number (0–255)

```lua
local flags = fget(1)   -- e.g. 3 means flags 0 and 1 are set
```

## Flag Index vs Bit Value

When calling `fget`/`fset` **with** an index, use **0–7**:

| Flag index | Bit value |
|------------|-----------|
| 0          | 1         |
| 1          | 2         |
| 2          | 4         |
| 3          | 8         |
| 4          | 16        |
| 5          | 32        |
| 6          | 64        |
| 7          | 128       |

Bit values only matter when reading/writing the whole bitfield (no index argument).

## Practical Example — Tile collision using flags

Set flag 0 in the sprite editor for all solid tiles, then:

```lua
function is_solid(tx, ty)
  return fget(mget(tx, ty), 0)
end
```

You can also use the `layers` parameter of `map()` to draw only sprites that have a specific flag set:

```lua
map(0, 0, 0, 0, 128, 64, 0x1)  -- only draw sprites with flag 0 set
```
