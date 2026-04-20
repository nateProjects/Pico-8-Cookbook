## Display Sprite

### Example 1 - Draw sprite (simple)

`spr( number, x, y )`

Eg. `spr(1,10,10)`

### Example 2 - Draw larger sprite and flip it (advanced)

`spr( number, x, y, width, height, flip_x, flip_y )`

Eg. (a 16x16 sprite facing left)- `spr(1,10,10,2,2,true,false)`

* width - number of sprites wide
* height - number of sprites high
* flip_x - facing left or right
* flip_y - facing up or down

### Example 3 - Stretch / scale a sprite region

`sspr( sx, sy, sw, sh, dx, dy, [dw, dh], [flip_x], [flip_y] )`

Stretches a rectangle of the sprite sheet to a destination rectangle on screen. Coordinates are in pixels.

* `sx, sy` — top-left pixel of the source region on the sprite sheet
* `sw, sh` — width and height of the source region in pixels
* `dx, dy` — destination position on screen
* `dw, dh` — destination size (optional; defaults to `sw, sh` — set these to scale)

Eg. (draw a 16×16 region starting at sprite-sheet pixel 0,0, scaled up to 32×32):

`sspr(0, 0, 16, 16, 10, 10, 32, 32)`

*Note: sprites 0–127 are in the main sprite sheet. Sprites 128–255 share memory with the bottom half of the map — use one or the other, not both.*

### Example 3 - Change sprite on map - 

    mset(1,1,1)
    cls()
    map(0,0,0,0,8,8)
    printh(mget(1,1)) -- print value to console
    mset(1,1,0) -- change to 0
    cls()
    map(0,0,0,0,8,8)
    printh(mget(1,1)) -- it's changed
