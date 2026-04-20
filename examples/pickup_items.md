# Picking Up Items

Store items in a table. Each frame, check whether the player overlaps any item. On overlap, apply the item's effect and remove it from the table with `del()`.

---

## Basic Example

```lua
player = {x=60, y=60, hp=3, score=0}

-- each item has a position, sprite, and type
items = {
  {x=20, y=48, spr=3, kind="coin"},
  {x=72, y=16, spr=4, kind="heart"},
  {x=96, y=80, spr=5, kind="coin"},
}

function _update()
  -- move player
  if btn(0) then player.x -= 2 end
  if btn(1) then player.x += 2 end
  if btn(2) then player.y -= 2 end
  if btn(3) then player.y += 2 end

  -- check all items for overlap with player
  for item in all(items) do
    if overlaps(player, item) then
      collect(item)
      del(items, item)
    end
  end
end

-- simple 8x8 box overlap check
function overlaps(a, b)
  return abs(a.x - b.x) < 8
     and abs(a.y - b.y) < 8
end

function collect(item)
  if item.kind == "coin" then
    player.score += 10
  elseif item.kind == "heart" then
    player.hp = min(player.hp + 1, 5)
  end
  sfx(0)  -- play pickup sound (assign one in the sfx editor)
end

function _draw()
  cls()
  for item in all(items) do
    spr(item.spr, item.x, item.y)
  end
  spr(1, player.x, player.y)
  print("score:"..player.score, 2, 2, 7)
  print("hp:"..player.hp, 2, 8, 8)
end
```

---

## Spawning Items at Runtime

Add items dynamically — useful for drops from defeated enemies.

```lua
function drop_item(x, y)
  -- 50% chance to drop a coin
  if rnd(1) > 0.5 then
    add(items, {x=x, y=y, spr=3, kind="coin"})
  end
end
```

---

## Items That Fall

Combine with gravity so dropped items settle on the ground.

```lua
function drop_item(x, y)
  add(items, {x=x, y=y, spr=3, kind="coin", dy=0})
end

-- in _update, before the overlap check:
for item in all(items) do
  if item.dy then
    item.dy = min(item.dy + 0.4, 4)
    item.y  += item.dy
    -- simple floor stop (or use solid() from jumping.md for tile collision)
    if item.y > 112 then
      item.y  = 112
      item.dy = 0
    end
  end
end
```

---

## Notes

- `del(items, item)` removes the item by value, not by index. Safe to call inside a `for item in all(items)` loop.
- The `overlaps` function above uses a loose 8px threshold. For tighter pickup detection on smaller sprites, reduce the threshold (e.g. `< 5`).
- To prevent re-collecting the same item in one frame (e.g. two checks in one update), the `del` call handles it — once removed, the item won't be in the table on the next iteration.
