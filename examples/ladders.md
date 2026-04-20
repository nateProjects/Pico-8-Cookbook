# Ladder Climbing

Ladders override gravity and let the player move freely up and down. The player enters ladder mode when their centre overlaps a ladder tile, and exits by jumping or walking off the top.

Set your tile flags in the sprite editor:
- **Flag 0** — solid wall/floor
- **Flag 1** — ladder

---

## How It Works

Each frame, check whether the player's centre tile is a ladder. If yes, cancel gravity and allow vertical movement with UP/DOWN. If no, normal gravity and jumping apply.

---

## Example

```lua
-- flag 0 = solid, flag 1 = ladder (set in sprite editor)

player = {x=60, y=40, dy=0, on_ground=false, on_ladder=false}

function _update()
  -- tile at the centre of the player
  local cx = flr((player.x+4)/8)
  local cy = flr((player.y+4)/8)
  player.on_ladder = fget(mget(cx, cy), 1)

  if player.on_ladder then
    -- on ladder: no gravity, up/down controls the player directly
    player.dy = 0
    if btn(2) then player.y -= 2 end  -- climb up
    if btn(3) then player.y += 2 end  -- descend

    -- jump off the ladder
    if btnp(4) then
      player.on_ladder = false
      player.dy = -4
    end
  else
    -- normal physics when not on a ladder
    if btnp(4) and player.on_ground then
      player.dy = -4.5
    end

    player.dy = min(player.dy + 0.4, 4)
    player.y += player.dy

    player.on_ground = false
    if player.dy >= 0 and
      (solid(player.x+1, player.y+8) or solid(player.x+6, player.y+8)) then
        player.y = flr(player.y/8)*8
        player.dy = 0
        player.on_ground = true
    end

    if player.dy < 0 and
      (solid(player.x+1, player.y) or solid(player.x+6, player.y)) then
        player.y = flr(player.y/8)*8 + 8
        player.dy = 0
    end
  end

  -- horizontal movement always applies
  if btn(0) then player.x -= 2 end
  if btn(1) then player.x += 2 end
end

function solid(px, py)
  return fget(mget(flr(px/8), flr(py/8)), 0)
end

function _draw()
  cls()
  map()
  spr(1, player.x, player.y)
end
```

---

## Notes

- **Entering from below**: if the ladder tile sits above a solid floor, the player walks to the base of the ladder and presses UP — `cy` will shift into the ladder tile on the next frame and switch modes automatically.
- **Reaching the top**: when the player climbs past the last ladder tile, `on_ladder` becomes false and normal physics resume. Place a solid floor tile at the top if you want them to land cleanly.
- **Ladder + platform**: if you want the player to stand on ladder tiles without actively climbing (idle grip), keep `dy = 0` whenever `on_ladder` is true and no vertical button is pressed — which the example above already does.
