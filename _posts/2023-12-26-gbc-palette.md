---
layout: post
published: true
date: 2023-12-26 00:00:00 -0500
title: Game Boy Color Palettes
tags: ["Game Boy"]
---
I'd managed to get colour working on the GBC using [GDBK-2020](https://github.com/gbdk-2020/gbdk-2020/)
but for some reason I ran into a mental block setting the map attributes. I also
had trouble exporting the palette and map attributes from the popular
[Game Boy Tile Designer](https://www.devrs.com/gb/hmgd/gbtd.html) (GBTD) and
[Game Boy Map Builder](https://www.devrs.com/gb/hmgd/gbmb.html) (GBMB).

Source code from this post is on GitHub at [rprouse/GBC-Samples](https://github.com/rprouse/GBC-Samples/tree/main/GBC-Palette).

I won't go into detail on using GBTD and GBMB, but I will highlight some of the
export settings that I found non-intuitive.

## Game Boy Tile Designer

Start by setting the **Color set** to **Game Boy Color** then editing the
**Palettes...** in the **View** menu. Then create your tiles.

To export,

### Standard

![GBTD Export Standard](/assets/img/2023-12-26-gbtd-std.png)

- **Type** - GBDK C file ( *. c)
- **Format** - Gameboy 4 color

### Advanced

![GBTD Export Standard](/assets/img/2023-12-26-gbtd-adv.png)

- **Colors** - Include palette colors
- **CGB palettes** - Constant per entry

## Game Boy Map Builder

Set the map size to 32x32 or less and load the tiles. Design the map, then
**Export to...** using **GBDK C file ( *. c)**. Under the **Location format**
tab,

![GBMB](/assets/img/2023-12-26-gbmb.png)

Add two Location formats, one for the map and one for the attributes,

1. [Tile number: Low 8] - 8
2. [GBC BG Attribute] - 8

Then set the following values,

- **Map layout** - Rows
- **Plane count** - 2 planes (16 bits)
- **Plane order** - Planes are continues
- **Tile offset** - 0

## GDBK Code

Include the tile and map header that were exported and make sure your Makefile
builds the exported C files.

I could not get GBMB to export a proper palette array, only the constants, so I
created the array myself,

```c
const palette_color_t palettes[32] = {
    PaletteTestTilesCGBPal0c0, PaletteTestTilesCGBPal0c1, PaletteTestTilesCGBPal0c2, PaletteTestTilesCGBPal0c3,
    PaletteTestTilesCGBPal1c0, PaletteTestTilesCGBPal1c1, PaletteTestTilesCGBPal1c2, PaletteTestTilesCGBPal1c3,
    PaletteTestTilesCGBPal2c0, PaletteTestTilesCGBPal2c1, PaletteTestTilesCGBPal2c2, PaletteTestTilesCGBPal2c3,
    PaletteTestTilesCGBPal3c0, PaletteTestTilesCGBPal3c1, PaletteTestTilesCGBPal3c2, PaletteTestTilesCGBPal3c3,
    PaletteTestTilesCGBPal4c0, PaletteTestTilesCGBPal4c1, PaletteTestTilesCGBPal4c2, PaletteTestTilesCGBPal4c3,
    PaletteTestTilesCGBPal5c0, PaletteTestTilesCGBPal5c1, PaletteTestTilesCGBPal5c2, PaletteTestTilesCGBPal5c3,
    PaletteTestTilesCGBPal6c0, PaletteTestTilesCGBPal6c1, PaletteTestTilesCGBPal6c2, PaletteTestTilesCGBPal6c3,
    PaletteTestTilesCGBPal7c0, PaletteTestTilesCGBPal7c1, PaletteTestTilesCGBPal7c2, PaletteTestTilesCGBPal7c3,
};
```

I then used the following code to load the palettes, the tiles, the map and the
attributes,

```c
set_bkg_palette(0, 8, palettes);
set_bkg_data(0, 1, PaletteTestTiles);
set_bkg_tiles(0, 0, MapWidth, MapHeight, MapPLN0);
set_bkg_attributes(0, 0, MapWidth, MapHeight, MapPLN1);
```

Note that `MapPLN0` is the map data and `MapPLN1` are the map attributes.
`set_bkg_attributes` is just a thin wrapper around `set_bkg_tiles` that sets
the `VBK_REG` appropriately.

```c
VBK_REG = VBK_ATTRIBUTES;
set_bkg_tiles(x, y, w, h, tiles);
VBK_REG = VBK_TILES;
```
