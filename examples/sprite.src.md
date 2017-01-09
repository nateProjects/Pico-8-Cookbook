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

### Example 3 - Scale sprite

[Sspr Syntax](http://pico-8.wikia.com/wiki/Sspr)

See also [ZSpr Function](http://pico-8.wikia.com/wiki/Draw_zoomed_sprite_(zspr))

*Note: Map based sprites start at number ?*

### Example 3 - Change sprite on map - 

    mset(1,1,1)
    cls()
    map(0,0,0,0,8,8)
    printh(mget(1,1)) -- print value to console
    mset(1,1,0) -- change to 0
    cls()
    map(0,0,0,0,8,8)
    printh(mget(1,1)) -- it's changed
