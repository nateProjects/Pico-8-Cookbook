# Palette Tricks

PICO-8 has three palette layers you can manipulate at runtime.

---

## PAL — Swap Colours

`pal(c0, c1, [p])` replaces colour `c0` with `c1`. `p` selects which palette:

| p | Name | Effect |
|---|------|--------|
| 0 | Draw palette (default) | Affects subsequent `spr`, `map`, drawing calls |
| 1 | Display palette | Remaps the whole screen instantly, including already-drawn pixels |
| 2 | Secondary palette | Used with `FILLP()` patterns |

`pal()` with no arguments resets **all** palettes to defaults.  
`pal(p)` resets just one palette (0, 1, or 2).

---

## Recolouring a Sprite

Useful for team colours, enemy variants, hit flashes etc.

```lua
-- draw player sprite in red (colour 8) instead of blue (12)
function draw_player(x, y, team)
  if team == 2 then
    pal(12, 8)   -- swap blue for red in the draw palette
  end
  spr(1, x, y)
  pal()          -- always reset afterwards
end
```

You can swap multiple colours at once:

```lua
-- make an enemy look dark/shadowed
pal(7,  6)   -- white  -> light gray
pal(10, 9)   -- yellow -> orange
pal(11, 3)   -- green  -> dark green
spr(enemy.spr, enemy.x, enemy.y)
pal()
```

---

## Hit Flash (Invincibility Flicker)

```lua
function _draw()
  -- flash white every other frame during invincibility
  if player.inv_timer > 0 and player.inv_timer % 2 == 0 then
    for c = 1, 15 do pal(c, 7) end  -- remap all colours to white
  end
  spr(player.spr, player.x, player.y)
  pal()
end
```

---

## PALT — Transparency

By default only colour 0 (black) is transparent when drawing sprites. `PALT` changes this.

`palt(c, transparent)` — set whether colour `c` is drawn as transparent.

`palt()` resets to default (only colour 0 transparent).

```lua
-- make colour 8 (red) transparent instead of colour 0 (black)
palt(0, false)   -- make black opaque
palt(8, true)    -- make red transparent
spr(1, x, y)
palt()           -- reset
```

Useful when your sprite background is a colour other than black, or when you want to use black as a visible outline colour.

---

## Screen Fade with the Display Palette

The display palette (p=1) remaps colours on the whole screen immediately — even pixels already drawn. This makes it ideal for fade effects.

```lua
-- fade: 0 = fully black, 15 = normal colours
-- call this after drawing everything else
function apply_fade(fade)
  -- rough approximation: remap each colour to black below threshold
  local darks = {0,0,0,0, 0,5,5,5, 13,4,9,3, 2,2,9,6}
  for c = 0, 15 do
    pal(c, fade >= c and c or darks[c+1], 1)
  end
end
```

Or a simpler all-or-nothing approach:

```lua
-- flash the whole screen to white (e.g. explosion)
function screen_flash()
  for c = 0, 15 do pal(c, 7, 1) end
end

-- restore
function screen_normal()
  pal(1)  -- reset display palette only
end
```

---

## Night / Tint Effect

```lua
-- apply a blue night tint to the display palette
function set_night_mode()
  pal(7,  6,  1)   -- white      -> light gray
  pal(6,  5,  1)   -- light gray -> dark gray
  pal(10, 1,  1)   -- yellow     -> dark blue
  pal(11, 3,  1)   -- green      -> dark green
  pal(9,  4,  1)   -- orange     -> brown
end

-- restore normal colours
function clear_night_mode()
  pal(1)
end
```

---

## Notes

- Always call `pal()` after drawing with a modified draw palette (p=0), or every subsequent draw call will use the remapped colours.
- The display palette (p=1) persists until you reset it — you don't need to call it every frame.
- `PAL` can also take a table as its first argument to remap multiple colours at once: `pal({[12]=8, [9]=8})`.
