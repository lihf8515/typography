# Typography - Fonts, Typesetting and Rasterization.

Typography is pure nim implementation for font rasterization (letter drawing) and text typesetting (text layout). It does *not* relay on any external library such as FreeType, stb_truetype, pango or HarfBuzz.

It does relay on stb_image for png writing, but if you don't write pngs it would not be included.


# Font file formats:
* SVG fonts - Most features are supported.
* TTF fonts - Fair support. Most modern features are supported but font format came out in 1994 and has a bunch of formats for different OSes that are not supported.
* OTF fonts - Basic TTF outline support only. No support for CFF or SVG outlines.

# Basic usage

```nim
var font = readFontSvg("fonts/Ubuntu.svg")
font.drawText(image, vec2(10, 50), "The quick brown fox jumps over the lazy dog.")
```

![example output](tests/basicSvg.png?raw=true)

```nim
var font = readFontTtf("fonts/Ubuntu.ttf")
font.drawText(image, vec2(10, 50), "The quick brown fox jumps over the lazy dog.")
```

![example output](tests/basicTtf.png?raw=true)

```nim
font.size = 8
font.drawText(image, vec2(10, 10), "The quick brown fox jumps over the lazy dog.")
font.size = 10
font.drawText(image, vec2(10, 25), "The quick brown fox jumps over the lazy dog.")
font.size = 14
font.drawText(image, vec2(10, 45), "The quick brown fox jumps over the lazy dog.")
font.size = 22
font.drawText(image, vec2(10, 75), "The quick brown fox jumps over the lazy dog.")
```

![example output](tests/sizes.png?raw=true)

```nim
font.drawText(image, vec2(10, 10), readFile("examples/sample.ru.txt"))
```
![example output](tests/ru.png?raw=true)

# Dealing with Glyphs

```nim
font.getGlyphOutlineImage("Q")
```
![example output](tests/q.png?raw=true)

```nim
echo font.glyphs["Q"].path
```
```
M754,236 Q555,236 414,377  Q273,518 273,717  Q273,916 414,1057  Q555,1198 754,1198  Q953,1198 1094,1057  Q1235,916 1235,717  Q1235,593 1175,485  L1096,565  Q1062,599 1013,599  Q964,599 929,565  Q895,530 895,481  Q895,432 929,398  L1014,313  Q895,236 754,236  Z M1347,314  Q1471,496 1471,717  Q1471,1014 1261,1224  Q1051,1434 754,1434  Q458,1434 247,1224 Q37,1014 37,717  Q37,421 247,210  Q458,0 754,0  Q993,0 1184,143  L1292,35  Q1327,0 1376,0  Q1425,0 1459,35  Q1494,69 1494,118  Q1494,167 1459,201  Z
```

# Subpixel glyphs with subpixel layout:

Each glyphs can be rendered with a subpixel offset, so that it fits into the layout:

![example output](tests/subpixelpos.png?raw=true)

Note how many of the "o"s and "m"s are different from each other. This happens because spaces between letters are not an integer number of pixels so glyphs must be rendred shifted by fraction of a pixel.
Here is how glyph changes with different subpixel offsets:

![example output](tests/subpixelglyphs.png?raw=true)

Here is how glyph changes with different subpixel offsets.

# Comparison to different OSs.

At the large pixels sizes of fonts > 24px the fonts on most operating system looks nearly identical. But when you scale the font bellow 24px different OSes take different approaches.
* OSX - tries to render fonts most true to how font designer intended even if they look a blurry.
* Windows - tries to render fonts to a pixel grid making them look sharper.
* Linux - configurable and somewhere between the two.
* iOS, Android - it really does not matter how the font is rendered because its almost always above 24px because of high resolution screens phones have.

```nim
   var font = readFontSvg("fonts/DejaVuSans.svg")
   font.size = 11 # 11px or 8pt
   font.drawText(image, vec2(10, 15), "The quick brown fox jumps over the lazy dog.")
```

Typography renderer - this library (4x):

![example output](tests/scaledup.png?raw=true)

Window ClearType renderer (4x):

![example output](tests/notepadWindows.png?raw=true)

Apple Core Text renderer (4x):

![example output](tests/sketchMac.png?raw=true)

Paint.net renderer Windows (4x):

![example output](tests/paintNetWindows.png?raw=true)

Bohemian Sketch renderer OSx (4x):

![example output](tests/sketchMac.png?raw=true)


# Subpixel rendering is on its way out

About a decade ago Subpixel rendering significantly improved readability of fonts. It would leak a bit of color to the left and right of text because color pixels were not square. Back then the pixels were big and monitors followed predictable patterns first in CRTs then in LCDs.

![example output](https://upload.wikimedia.org/wikipedia/commons/5/57/Subpixel-rendering-RGB.png?raw=true)

Then everything changed. In 2018 our pixels are really small and they don't follow the same typical CRT or LCD orientation.

![example output](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4d/Pixel_geometry_01_Pengo.jpg/220px-Pixel_geometry_01_Pengo.jpg)

The fact that there is no standard pixel layout grid anymore and the fact that high resolution displays are everywhere makes subpixeling absolute. Apple, Adobe, Bohemian and other companies in the typography space are abandoning subpixeling for these reasons.

This library does not support Subpixel rendering.

[Neat tricks](http://www.typophile.com/node/60577) [with Subpixel rendering](http://www.typophile.com/node/61920)

# How to convert any font to SVG font using FontForge:

SVG fonts are really nice. The are simple to parse and understand and debug. They are very uncommon though. But they are good as a debug input or output or intermediate step.

```bash
$ fontforge -c 'Open($1); Generate($2)' foo.ttf foo.svg
```