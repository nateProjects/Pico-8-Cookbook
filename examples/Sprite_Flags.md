# Pico-8 Sprite Flags

    fset (sprite,flag)
eg. 

    fset (1,3) -- set flag to 3 (1+2)

| # | 1 | 2 | 4 | 8 | 16 | 32 | 64 | 128 |
| --- | --- | --- | --- | --- | --- | --- | --- |

| 1 | o |
| 2 || o |
| 3 | o | o |
| 4 ||| o |
| 5 | o || o |
| 6 || o | o |
| 7 | o | o | o |
| 8 |||| o |
| 9 | o |||o |
| 10 || o || o |
| 11 | o | o || o |
| 12 ||| o | o |
| 13 | o || o | o |
| 14 || o | o | o |
| 15 | o | o | o | o |
| 16 ||||| o |
| 32 |||||| o |
| 64 ||||||| o |
| 128 |||||||| o |
| 255 | o | o | o | o | o | o | o | o |

    mset (sprite, flag)
    
eg. 
   
    mset (2,5) -- set flag to 5 (1+4)

    if (mset(3,7)) -- if sprite 3 has flag of 7 then true
