
```
Usage: PatchWideScreen [-01234567] [iso]

if iso is not provided, If iso is not provided, ELF file named as 'SLPS_255.44' is supposed to be patched

all the options should be concatenated together

```

```
Options meaning
-0: wide screen, no -0 means no wide screen

-1: FMV's fix, no -1 means no FMV's fix, which is default

-2: Focus Effect Off, hence, no -2 means Focus Effect On, which is default

-3: Bloom offset (fixes bloom glitch)

-4: Dither + Ghost post-process Effect Off

-5: Disable dark filter (cutscene)

-6: Disable all bloom (speedup, but makes the game seem dull)

-7: Disable overbloom (cutscene) and Decrease overbloom (gameplay)
```

Most of the options are not recommended to open. According to the pnatch file included in pcsx2, the recommended options are:

`PatchWideScreen -0123 [iso]`