---
layout: post
title: Super Connard
subtitle: Super Asshole
tags: [gameboy, reverse, z80, radare2]
image: http://furrtek.free.fr/superconnard/logo.png
share-img: http://furrtek.free.fr/superconnard/logo.png
---

I've recently entered, as personal hobby, into the world of game development. And in 2020, what is more cool that trying to code on GameBoy™ platform.

Their is plenty of documentation and code samples out their and you can find a lot of references under [Awesome Game Boy Development](https://github.com/gbdev/awesome-gbdev).

During my learning I found and read a lot of documentation and one of them is [Tuto: Programmer sur Gameboy](http://furrtek.free.fr/?a=gbasm) (fr). This guy (alias [_Furrtek_](https://twitter.com/furrtek?lang=fr)) seems to know a lot about electronics and coding on hardware, by looking some of his work:
 - [GB303 wavetable-based TB-303 style synthesizer for the Nintendo Gameboy](http://furrtek.free.fr/gb303)
 - [TV adapter for Lynx console](https://mag.mo5.com/actu/58759/adaptateur-tv-les-25-ans-lynx) (fr)

But what caught my attention is [Super Connard™](http://furrtek.free.fr/superconnard/index.php) (fr) (translated as _Super Asshole_), which ROM is available, but:

**fr**:
> Comment peut-on y jouer, alors ?
>
> Comme promis, maintenant que les cartouches sont coulées, vous pouvez librement télécharger et partager le ROM.
A un détail près: il vous faudra faire sauter la protection anti-émulateur ! Et oui, le jeu porte bien son nom ! ;)

**en**:
> How can we play, then ?
>
> As promised, now that cartridges have been produced, you can freely download and share the ROM.
> At on small detail: you'll need to get over anti-emulator protection ! And yes, the game deservers its name ! ;)

So that sounds like a challenge. This game has been release in 2013 and its source have been since then already disclosed. Let's assume we do not have them !

**DISCLAIMER: The views and opinions expressed in _Super Connard™_ game are those of the authors and do not represent policy or position of _this_ blog post, which only purpose is to provide as educative methodology in binary analysis.**

# Tools
## Radare2

[radare2](https://github.com/radareorg/radare2) is an open-source UNIX-like reverse engineering framework and command-line toolset.

It allows basically to play with any kind of binary and analyse it. Since a Game Boy™ cartridge ROM is nothing more than some binary data, this should help us.

## GameBoy™ emulator

Any emulator should be able to be used.

# Let's go

At the time the game was released, it was only available on purchase of a physical Game Boy™ cartridge. There is a video showing how [Xylit0l](https://twitter.com/xylit0l?lang=fr) made a dump of ROM ([Gameboy cartridge dumping (Super Connard)](https://vimeo.com/290800284)).

Now that the ROM is available, just  get it: [sc_prod.gb](http://furrtek.free.fr/superconnard/sc_prod.gb).

Time to launch the game:
```bash
$> gngb -a sc_prod.gb
stream 5
Open file sc_prod.gb
Rom Name sc_prod
Name    : SUPERCONNARD
Normal GameBoy
configuration 00 : ROM ONLY
ROM size 0 : 2 * 16 kbyte
NO RAM
Country : Non-Japanese
```
and a window opens !

![Fuck you, emulator!](/img/super_connard-fuck_you.png)

Dude, that's so rude.

What does this file contains
```bash
strings sc_prod.gb | less
...
Fuck you,
emulator!
...
Code/Graphismes:
         Furrtek
Musique/FX:
       Don Ratta
Remerciements:
            Dyak
         ElBarto
 Marat Fayzullin
    Martin Korth
       Nitro2k01
           Orion
       RayXambeR
        Realmyop
         Robotwo
         Scruffy
     Ville Helin
...
```
at least, now, we know who insulted us.

I assume that [radare2](https://github.com/radareorg/radare2) is already installed, otherwise feel free to download it [here](https://www.radare.org/r/down.html).

Now we open the binary with `radare2`; use `-w` since we might need to update some data.

```bash
$> radare2 -w sc_prod.gb
[0x00000100]> i~format
format   ningb
[0x00000100]> i~machine
machine  Gameboy
```
It's a GameBoy™ ROM. We already knew it. `0x100` is the start of any ROM.

```bash
[0x00000100]> px
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x00000100  00c3 5001 ceed 6666 cc0d 000b 0373 0083  ..P...ff.....s..
0x00000110  000c 000d 0008 111f 8889 000e dccc 6ee6  ..............n.
0x00000120  dddd d999 bbbb 6763 6e0e eccc dddc 999f  ......gcn.......
0x00000130  bbb9 333e 5355 5045 5243 4f4e 4e41 5244  ..3>SUPERCONNARD
0x00000140  0000 0000 3030 0000 0000 0133 00bf 7e70  ....00.....3..~p
0x00000150  f331 f4ff afe0 2621 00c0 01ff 1fcd 7300  .1....&!......s.
```

Bytes 0x104 to 0x130 are Nintendo® logo which are mandatory for any ROM that is meant to work on real GameBoy™.
```
;*
;* Nintendo scrolling logo
;* (Code won't work on a real GameBoy)
;* (if next lines are altered.)
NINTENDO_LOGO : MACRO
    DB  $CE,$ED,$66,$66,$CC,$0D,$00,$0B,$03,$73,$00,$83,$00,$0C,$00,$0D
    DB  $00,$08,$11,$1F,$88,$89,$00,$0E,$DC,$CC,$6E,$E6,$DD,$DD,$D9,$99
    DB  $BB,$BB,$67,$63,$6E,$0E,$EC,$CC,$DD,$DC,$99,$9F,$BB,$B9,$33,$3E
ENDM
```
Source: https://github.com/gbdev/hardware.inc

Conventionally, GameBoy™ program starts at `0x150`. Let's confirm it.
```bash
[0x00000100]> pd 70
            ;-- pc:
/ (fcn) entry0 54
|   entry0 ();
|           0x00000100      00             nop
|       ,=< 0x00000101      c35001         jp main
        |   0x00000104      ceed           adc 0xed
...
|      ||   ;-- main:
|      ||   ; CODE XREF from entry0 (0x101)
|      |`-> 0x00000150      f3             di
|      |    0x00000151      31f4ff         ld sp, 0xfff4
```

Or we could simply just have jumped to `main`
```
[0x00000000]> s main
[0x00000150]>
```

Looking for `Fuck you` string leads us to `0x4ef`
```bash
[0x00000100]> izzq~Fuck
0x4ef 10 9 Fuck you,
```

So lets just look for where this string is loaded
```bash
[0x00000100]> pd 5000 | grep 0x04ef
            0x000007f9      11ef04         ld de, 0x04ef
```

Jump few instruction before
```
[0x00000100]> 0x000007f9-50
[0x000007f9]> pd
            0x000007c7      3ee4           ld a, 0xe4
            0x000007c9      e047           ld [0xff47], a
            0x000007cb      3e91           ld a, 0x91
            0x000007cd      e040           ld [0xff40], a              ; LCDC
       ,.-> 0x000007cf      76             halt
       |:   0x000007d0      00             nop
       ``=< 0x000007d1      18fc           jr 0xfc
            0x000007d3      43             ld b, e
            0x000007d4      68             ld l, b
            0x000007d5      65             ld h, l
            0x000007d6      63             ld h, e
            0x000007d7      6b             ld l, e
            0x000007d8      73             ld [hl], e
            0x000007d9      75             ld [hl], l
            0x000007da      6d             ld l, l
        ,=< 0x000007db      2065           jr nZ, 0x65
        |   0x000007dd      72             ld [hl], d
        |   0x000007de      72             ld [hl], d
        |   0x000007df      6f             ld l, a
        |   0x000007e0      72             ld [hl], d
        |   0x000007e1      ff             rst sym.rst_56
        |   ; CALL XREF from entry0 (0x16f)
        |   0x000007e2      f3             di
        |   0x000007e3      cdc400         call 0x00c4
        |   0x000007e6      cd7e03         call 0x037e
        |   0x000007e9      cdc801         call 0x01c8
        |   0x000007ec      3ee4           ld a, 0xe4
        |   0x000007ee      e047           ld [0xff47], a
        |   0x000007f0      210040         ld hl, 0x4000
        |   0x000007f3      110080         ld de, 0x8000
        |   0x000007f6      cd1404         call 0x0414
        |   0x000007f9      11ef04         ld de, 0x04ef
```
and jump to where we are called from: `0x16f`
```
[0x000007c7]> 0x16f
[0x0000016f]> pd
|           0x0000016f      c4e207         call nZ, 0x07e2
|           0x00000172      cdea05         call 0x05ea
|           0x00000175      cd8201         call 0x0182
|           0x00000178      fe04           cp 0x04
|           0x0000017a      cc660d         call Z, 0x0d66
|           0x0000017d      cd2008         call 0x0820
\       @-> 0x00000180      18fe           jr 0xfe
            ; CALL XREF from entry0 (0x175)
```
We see that we jump to code with `call nZ, 0x07e2`, so let's try to invert the condition. For that, we need to modify z80 opcode: here `c4` by `cc`.

```
CALL NZ,nn	17/10	18/11	5/3	8/7/3²	C4 nn nn	3
CALL Z,nn	17/10	18/11	5/3	8/7/3²	CC nn nn	3
```
Source: http://map.grauw.nl/resources/z80instr.php

Enter visual mode, and press `i` to update first `c4` by `cc`. Press `q` to quit editing.
```
[0x0000016f]> v
[0x0000016f [Xadvc]0 0% 1728 sc_prod2.gb]> xc @ main+31 # 0x16f
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF  comment
0x0000016f  c4e2 07cd ea05 cd82 01fe 04cc 660d cd20  ............f..
[0x0000016f]> pd1
            0x0000016f      cce207         call Z, 0x07e2
```

And start the ROM in emulator...

[Checksum error!](/img/super_connard-checksum_error.png).

Damned ! It seems that there is some checksum protection.

Try again.

```
$> radare2 -w sc_prod.gb
[0x00000100]> izzq~Checksum
0x7d3 15 14 Checksum error
[0x00000100]> pd 5000 | grep 7d3
            0x000007b7      11d307         ld de, 0x07d3
[0x00000100]> 0x000007b7-50
[0x00000785]> pd
            0x00000785      7e             ld a, [hl]
            0x00000786      110000         ld de, 0x0000
        .-> 0x00000789      7b             ld a, e
        :   0x0000078a      86             add [hl]
        :   0x0000078b      5f             ld e, a
       ,==< 0x0000078c      3001           jr nC, 0x01
       |:   0x0000078e      14             inc d
       `--> 0x0000078f      23             inc hl
        :   0x00000790      0b             dec bc
        :   0x00000791      78             ld a, b
        :   0x00000792      b1             or c
        `=< 0x00000793      20f4           jr nZ, 0xf4
            0x00000795      214d00         ld hl, 0x004d               ; 'M'
            0x00000798      2a             ldi a, [hl]
            0x00000799      bb             cp e
        ,=< 0x0000079a      c2a207         jp nZ, 0x07a2
        |   0x0000079d      7e             ld a, [hl]
       ,==< 0x0000079e      c2a207         jp nZ, 0x07a2
       ||   0x000007a1      c9             ret
       ``-> 0x000007a2      cd7303         call 0x0373
            0x000007a5      af             xor a
            0x000007a6      e040           ld [0xff40], a              ; LCDC
            0x000007a8      210040         ld hl, 0x4000
            0x000007ab      110080         ld de, 0x8000
            0x000007ae      cd1404         call 0x0414
            0x000007b1      cd7e03         call 0x037e
            0x000007b4      cdc801         call 0x01c8
            0x000007b7      11d307         ld de, 0x07d3
```
Let's see if we can prevent code from reaching `0x000007a2` instruction. This time replace `c2` by `ca` (ie `jp nz` by `jp z`)
```
        ,=< 0x0000079a      c2a207         jp nZ, 0x07a2
        |   0x0000079d      7e             ld a, [hl]
       ,==< 0x0000079e      c2a207         jp nZ, 0x07a2
```
in order to have
```
[0x0000079a]> pd
        ,=< 0x0000079a      caa207         jp Z, 0x07a2
        |   0x0000079d      7e             ld a, [hl]
       ,==< 0x0000079e      caa207         jp Z, 0x07a2
```

Exit `radare2`.

Start the ROM in emulator, and cross fingers...

Voilà !

![Super connard](video/super_connard.webm)

# References
 - [Reverse engineering a Gameboy ROM with radare2](https://www.megabeets.net/reverse-engineering-a-gameboy-rom-with-radare2)
 - [Super Connard (GameBoy) - rgcd.co.uk](https://www.rgcd.co.uk/2013/06/super-connard-gameboy.html)
 - [Gameboy cartridge dumping (Super Connard) - Xylitol](https://vimeo.com/290800284)
 - [Z80 / R800 instruction set](http://map.grauw.nl/resources/z80instr.php)
