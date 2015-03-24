changes in this fork
--------------------

For a project, I needed to convert a bdf font to a C array.  I needed a specific bit order and rotation.  I also wanted to truncate the last byte since it was always 0x00 for all glyphs. 

Also, I wanted it to run on MacOS, so I modified the Makefile and some of the source files to compile appropriately.

This is not meant to be well tested or bullet-proof.  I pretty much just hacked in support for the things I needed.  I make no claims of it being of any use to anyone else.  Use it at your own risk. 

original README follows.

bdfe
----

**bdfe** might sound like crazy alphabet mixture but actually it is short name for [Bitmap Distribution Format Exporter](http://en.wikipedia.org/wiki/Glyph_Bitmap_Distribution_Format "bdf wiki") - utility which can extract subsets or export all glyphs from .bdf font files.

Why? I've decided to add a small oled display to my temperature/humidity station based on modded [MMR-70 FM Music Transmitter](https://github.com/achilikin/mmr70mod "MMR70 mod") but could not find any free 8x8 and 8x16 fonts in C format. There are tons of free fonts in bdf format because for years bdf has been used by UNIX X Windows. Converters available as well, but I needed very specific converter which would create fonts for SSD1306 based OLED screens.

So here it is - bdf font converter which can export glyphs from bdf and present them as array of bytes. It also can rotate 8x8 and 8x16 glyphs CCW so they can be used on displays with SSD1306 controller.

I'm running it with my Raspberry Pi (same RPi I use to program MMR70 ATmega) but it should run on any other Linux machine as well.


Command line options
--------------------

bdfe understands a few command line options, in plain English words or, for lazy people like me, in short single letter option which happened to be the first letter of the corresponding word. 

```
bdfe [options] <bdf file>
  options are:
  header:     print file header
  verbose:    add extra info to the header
  line:       one line per glyph
  subset a-b: subset of glyphs to convert a to b, default 32-126
  all:        print all glyphs, not just 32-126
  native:     do not adjust font height 8 pixels
  ascender H: add extra ascender of H pixels per glyph
  rotate:     rotate glyphs' bitmaps CCW
  display A:  show converted font on SSD1306 compatible display
              using I2C bus 1, hexadecimal address A (default 3C)
  updown:     display orientation is upside down
```

So ```bdfe header verbose line all native file.bdf``` and ```bdfe -h -v -l -a -n file.bdf``` are the same.

There is no output file option - just redirect bdfe output to a file: ```bdfe -h -v -l -a -n font.bdf > font.h```
 
OLED display is not needed for conversion but useful to have as you can immediately see if font is suitable with under-/over-line or inverse attributes. By default I2C slave address x3C is used, on my display this address is selected by connecting DC to GND, connecting it to Vcc switches address to x3D. If you do not know address of your SSD1306 display use ```i2cdetect  

Source code
-----------

Source code can be split to standalone modules which might be used in other projects separately:

```bdfe.h, bdfe.c``` - BDF file parser, single (quite big) function.
```rterm.h, rterm.c``` - raw terminal input, kind of old nice getch().
```ossd_i2c.h, ossd_i2c.c``` - OLED SSD1306 controller interface using I2C bus. Simple text mode only with direct access to SSD1306 registers, no shadow graphics buffer - memory on MMR70 ATmega32 MCU is a precious resource and anyway MMR70 will update screen only once in a while. Currently only fonts 8 and 16 bits high are supported. Underline, overline and inverse attributes can be used. 
```pi2c.h, pi2c.c``` - basic I2C wrapper for i2c-dev library, allows to write to SSD1306 connected to I2C bus.
```font8x8.h, font8x16.h``` - converted bdf files.
```main.c``` - puts all of above together and does the work.

Examples
--------
![bdfe screen](http://achilikin.com/github/bdfe-01.png)
Converting 7x8 font... Use ```bdfe ... | less``` to scroll exported font up and down.

![8x8 screen](http://achilikin.com/github/font8x8.gif) ![8x16 screen](http://achilikin.com/github/font8x16.gif) ![mmr70 screen](http://achilikin.com/github/mmr70.png)

OLED screen with 8x8 and 8x16 fonts converted and screen connected to a modded version of [MMR-70](https://github.com/achilikin/mmr70mod") with current temperature and humidity reading.
For 8x16 font command option ```ascender 1``` was used to lower glyphs 1 pixel down.
  

Licence
-------
BSD
 

