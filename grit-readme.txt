
         -------------------------------------------------------
       //                                                       \\
      ||         GG  GGG                                         ||
      ||        G G  GG G                                        ||
      ||          G  GG G                                        ||
      ||          GGGGGG      rrrr     i   i     ttt             ||
      ||           G GG      r    r    i  i     t   t            ||
      ||          G  GG      r r  r     ii      t   t            ||
      ||         G   GG       r   r    i i      tttt             ||
      ||                               i i      t                ||
      ||                               ii       t                ||
      ||                                                         ||
      ||=========================================================||
      ||         GRIT : GBA Raster Image Transmogrifier          ||
      ||                  by : Jasper Vijn (cearn)               ||
      ||                            v0.8                         ||
      ||                 (last update: 2008-03-04)               ||
      ||                   (started: 2005-09-15)                 ||
       \\_______________________________________________________//


// --------------------------------------------------------------------
// Introduction
// --------------------------------------------------------------------

Grit and Wingrit are a pair of converters from PC-based bitmaps to
GBA readable graphic formats. The functionality is (roughly) the 
same as the exporter found in my bitmap editor Usenti, but these 
stand-alone versions can handle more file types by using the 
FreeImage graphics library (freeimage.sourceforge.net/).

(Win)grit converts bitmaps into palettes, graphic data (tile or 
bitmap formats), tilemaps and metamaps. The output is in C arrays, 
asm arrays, binary files or a GBFS archive (tepples' GameBoy File 
system, see www.pineight/com). (Win)grit supports the three
main BIOS compression formats (LZ77, huffman, RLE), as well as 
graphics bitpacking to allow graphic data of 1,2,4,8 and 16 bpp. 
The data arrays can be in byte, halfword or word formats and a
header file can be created with the declarations of these. The 
tilemapping option includes unique tile reduction, different 
map layouts and even metamapping with variable width and heights. 
This can also be used for 1D sprite-sheet conversions.

NOTE: (Win)grit was born '(Win)git. But because the linux kernel 
versioning is also called 'git' and g(r)it's supposed to be a
multi-platform too, it had to be renamed. Hence, 'grit'.


--- Grit ---

Grit is the command-line version, the usage and option list is 
given below. For the basic CLI functionality I am indebted to
gauauu (www.tolberts.net), who basically handed me a set of CLI 
functions, which proved a _lot_ easier to use than what I had 
initially planned on using. Several others (in particular 
Dave Murphy (www.devkitpro.org)) have also been helpful in making 
this a multi-platform tool.

for making this multi-platform.

- cearn (cearn@coranac.com)


--- Changes ---

* 20080304, v0.8
  - Added shared palette and graphics options ('-pS' and '-gS').
  - Added a separate run for shared data; this also disables items 
    from earlier exports if they're supposed to be shared.
  - New filetype: GRF ('-ftr'). GRF is a RIFF with chunks for graphics,
    map, metamap and palete data and a header with additiona information. 
    A reader for it can be found in the docs. GRFs are also available 
    for array-data using '-fr'.
  - Added a 'fake' compression option ('-Z0') to make non-compresssed
    data follow the same format as compressed data so that a uniform loader
    can be made.
  - Put the version and completion info under the logging to keep the 
    shell clean.
  - Exit code for main (0 for OK, 1 for FAIL).
  - Fixed stupid RGB/BGR bug in the conversion code.
* 20070403, v0.7
  - Name change to 'grit'.
  - Changed the way directories work. All relative paths start from 
    the current working directory, whatever that may be. This also 
	fixes a bug when for dir-less -fx options (thanks dovoto).
  - Default filetype is now assembly.
  - Added -We, -Ww, -Ws aliases for -W<n>.
  - Added no converted output option -ft!.
  - Added batchfile 'grind.bat' for desktop Drag&Drop runs, for when 
    you want a simple&quick conversion.
* 20070317, v0.7b
  - Batchrun: Multiple bitmaps in one go (no wildcards though).
  - Dwimming for transparency options (i.e., -gT and -pT interaction)
  - Error/warning/status logging functions (-W).
  - external tilefile support (-fx).
  The batchrun and external tilefile support is still experimental. 
  YHBW.
* 20070103, v0.6.2
  - true -> pal resulted in a zero-entry palette. This is now fixed. 
    Thanks wintermute.
* 20061208, v0.6.1
  - Fixed -gT use, which was very, very broken. There should be some 
    interaction between -gT and -pT, but not sure which yet.
  - The append modes for C and asm should work properly now, i.e., 
    if the array already exists the new ones will replace it.
* 20060803, v0.6: 
  - Thanks to wintermute, the code should be fully multiplatform now.
  - Added -pT, -gT, -ff options. See usage.


// --------------------------------------------------------------------
// Basic Grit usage
// --------------------------------------------------------------------

NOTE: this is just a simple readme. For details, see the htm files.

The basic format for using grit is:

   grit srcfiles [opts]

That is, `program name', source bitmap(s) followed by any combination 
of options. Example for conversion of foo.bmp into a set of 16x16p 
4bpp sprites, using only the first 16 palette entries, to 16bit C 
arrays:

    grit foo.bmp -Mw 2 -Mh 2 -gB4 -pe 16 -U16 -ftc

Grit is relatively lenient in terms of what you enter. For one, it
searches out _correct_ options, so that incorrect or options are 
simply ignored, as are later version of option types in the case of 
multiple instances. Most options consist of a `base' and a `value'; 
the space between the two is optional. For example, the bitdepth 
option in the example ('-gB4') would also have been excepted as 
'-gB 4'. There is also a complex system of defaults to simplify the 
input. These are given in brackets in the list below. but for easy 
reference, here 's a list:

Palette: full source palette, u16 C array
Graphics: full image, original bpp (kinda). Paletted images default 
  to tiles, true-color will be 16bpp bitmaps. u32 C arrays
Map: no map. But if you have any map options (`-m*'), it'll default to 
  a flat regular tilemap, reduced for tiles and flips. u16 C array
Misc: no compression, create header, symbol name derived from 
  destination filename, which is derived from the source filename.


--- Palette options (base: "-p") ---
-p | -p!       Include or exclude pal data [inc]
-pu(8|16|32)   Pal data-type: u8, u16 , u32 [u16]
-pz[!lhr0]     Pal compression: off, lz77, huff, RLE, fake [off]
-ps{n}         Pal range start [0]
-pe{n}         Pal range end (exclusive) [pal size]
-pn{n}         Pal count [pal size]. Overrides -pe
-pS            NEW: shared palette
-pT{n}         Transparent palette index; swaps with index 0 [0]

--- Graphics options (base: "-g") ---
-g | -g!       Include  or exclude gfx data [inc]
-gu(8|16|32)   Gfx data type: u8, u16, u32 [u32]
-gz[!lhr0]     Gfx compression: off, lz77, huff, RLE, fake [off]
-ga{n}         Gfx pixel offset (non-zero pixels) [0]
-gb | -gt      Gfx format, bitmap or tile [tile]
-gB{n}         Gfx bit depth (1, 2, 4, 8, 16) [img bpp]
-gS            NEW: Shared graphics
-gT{n}         Transparent color; rrggbb hex or 16bit BGR hex [FF00FF]

-al{n}         Area left [0]
-ar{n}         Area right (exclusive) [img width]
-aw{n}         Area width [img width]. Overrides -ar
-at{n}         Area top [0]
-ab{n}         Area bottom (exclusive) [img height]
-ah{n}         Area height [img height]. Overrides -ab

--- Map options (base: "-m") ---
-m | -m!       Include or exclude map data [exc]
-mu(8|16|32)   Map data type: u8, u16, u32 [u16]
-mz[!lhr0]     Map compression: off, lz77, huff, RLE, fake [off]
-mR{t,p,f}     Tile reduction: (t)iles, (p)al, (f)lipped
                 options can be combined [-mRtpf]
-mR[48a]       Common tile reduction combos: reg 4bpp (-mRtpf), 
                 reg 8bpp (-mRtf), affine (-mRt), respectively
-mR!           No tile reduction (not advised)
-mL[fsa]       Map layout: reg flat, reg sbb, affine [reg flat]
--- Meta/Obj options (base: "-M") ---
-Mh{n}         Metatile height (in tiles!) [1]
-Mw{n}         Metatile width (in tiles!) [1]
-MRp           Metatile reduction (pal only) [none]

--- File / var options ---
-ft[!csbgr]    File type (no output, C, GNU asm, bin, gbfs, grf) [.s]
-fr            NEW: Enable GRF-format for .c or .s
-fa            File append
-fh | -fh!     Create header or not [create header]
-ff{name}      Additional options read from flag file [dst-name.grit]
-fx{name}      External tileset file
-o{name}       Destination filename [based on source]
-s{name}       Symbol base name [based from dst]
-O{name}       NEW: Destination file for shared data
-S{name}       NEW: Symbol base name for shared data

--- Misc ---
-q             Quiet mode; no report at the end
-U(8|16|32)    All data type: u8, u16, u32
-W{n}          Warning/log level 1, 2 or 3 [1]
-Z[!lhr0]      All compression: off, lz77, huff, RLE, fake [off]

New options: -fr, -ftr, -gS, -O, -pS, -S, -Z0 (et al)

If you explicitly mention a destination file during a multi-source run,
be sure to add the -fa flag and DO NOT use the -s flag. In a shared-data 
run, the shared filename and symbol name can be controlled with

The external tilefile is experimental too. It seems to work well 
enough, but just to be sure, always start with an 8bpp bitmap that's 
already tiled (or non-existent), and use either bmp, gif or png. Pcx 
support will be added later (FreeImage doesn't do pcx writing. That's 
why). I'll try to add metatiling to it too at some point.


// --------------------------------------------------------------------
// Examples for common conversions
// --------------------------------------------------------------------

Mode 0 map (=regular), 4bpp tiles, tile/pal/flip reduced, sbb format
  -gt -gB4 -mRtpf -mLs
or
  -gB4 -mR4 -mLs

Mode 2 map (=affine), tile reduced, map in bytes
  -gt -gB8 -mRt -mLa -mu8

Mode 3 or 5 bitmap:
  -gb -gB16

Mode 4 bitmap, forced to screen size
  -gb -gB8 -aw 240 -ah 160

Sprite, 4bpp, 16x16p (=2x2t)
  -gt -gB4 -Mw2 -Mh2

Sprite, 4bpp using pal16-bank 4 (colors 0x40 - 0x4F)
  -gt -gB4 -ps 0x40 -pn 16
or
  -gt -gB4 -ps 0x40 -pe 0x50


Mode 0 metamap, 4x3 metatiles, full tileset reduction
  -gt -gB4 -mR4 -Mw 4 -Mh 3 

NDS 16bpp bitmap, with cyan as transparent color
  -gb -gB16 -gT 00FFFF

Shared data: multi-bitmap to tilemaps + single tileset conversion.
If you want to convert a number of bitmaps to a single tileset and 
multiple maps, use -gS. If you want to save the tileset, or want to 
use a pre-made tileset, use -fx.

The output of this example is a single file with the combined tileset 
as sharedTiles, its palette as sharedPal and the separate maps as 
a1Map, a2Map, a3Map.
[code]
GRIT	:= grit.exe
BMPS	:= a1.bmp a2.bmp a3.bmp
TILESET	:= tiles.bmp

tilemap.s : $(BMPS)
	$(GRIT) $^ -o $@ -fa -gS -Sshared -mR8
[/code]

