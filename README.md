# FX3-SOLUTIONS

Notes and material for Far Cry 3 sandbox / map-editor modding, written for the
problems that are not documented anywhere because the editor shipped in 2012 and
nobody has picked the engine apart since.

First topic: **why the object browser shows empty tiles, and what a thumbnail
actually is.**

---

## Read this first

Far Cry 3 and everything inside it belongs to Ubisoft. The author owns none of
it. [LICENSE](LICENSE) is split so that the boundary is explicit — Part A is
what is licensed, Part B is what is not. The short version: the text and format
notes here are yours to reuse non-commercially; the game assets are not, and the
`thumbnails/` folder in this repository is Ubisoft-derived imagery you should
treat as reference material, not as content you get rights to.

---

## The problem

You open the Far Cry 3 in-game editor, browse an object folder, and the grid is
almost entirely blank. Not broken — blank. You hover a slot and it either paints
nothing or paints the same generic glyph.

It is not caused by a bad map, a bad install, or missing textures in your `.fc3map`.
It is how the editor's icon lookup was designed to fail when the baked icon store
is absent — see the next section.

## How the editor finds an icon

Two lookups happen, in this order, for every library entry you browse:

| # | Path | Who writes it | What it holds |
|---|------|---------------|---------------|
| 1 | `<editor data>\ingameeditor\thumbnails\pc\<id>.xbt` | **Nobody, in a Steam install** | Baked DXT5 icons — the store the vendor shipped on console/disc builds |
| 2 | `<editor data>\ingameeditor\thumbnails\png_src\<id>.png` | `FC3.dll`, as a side effect of any live render | Full-resolution tight-cropped PNG of whatever was just rendered |
| 3 | `<editor install>\bin\thumbnails\<key>.png` | The decompiled editor build, at runtime | 128x128 normalised PNG cache |

Lookup 1 is the intended path. `FC3.dll` carries the format string
`ingameeditor\thumbnails\pc\%u%s.xbt` (and `pc\col%u.xbt`, `pc\wat%u.xbt`, plus a
`console\` mirror of the whole set), so every icon in the game normally arrives as a
pre-baked `.xbt`. In a Steam installation that folder does not exist. There is no
`pc\*.png` variant of the format string either — the `pc` store only accepts `.xbt`.

Lookup 2 only ever contains things you have already rendered in this session, and
lookup 3 only contains what the editor managed to render before you stopped looking.

So with no baked store and nothing cached, the browser is blank, and it stays blank
until something renders.

## Why it looks worse now than in 2012

Ubisoft's own icon set was built for the stock library. Measured against the shipped
`ingameeditor` data: roughly **2,318 baked icons for roughly 2,336 stock objects** —
that is a complete set for the game as released.

Popular object packs expand that library to about **30,000 entries**, i.e. **~13x the
prop count with 1x the icon coverage**. Every prop the pack adds has no baked icon and
no cache entry, so it is blank by definition. Editors you see on YouTube that look
"full of icons" are either using the stock library or are the console build of the
editor, which does ship the `pc` store. This is worth knowing before you spend a day
chasing a bug that is not there.

## Making icons appear, and what it costs

The live renderer is the practical route: `FCE_ObjectRenderer_SetActive`,
`FCE_ObjectRenderer_RenderObject`, `FCE_ObjectRenderer_IsSnapshotReady`,
`FCE_ObjectRenderer_GetSnapshot`, `FCE_ObjectRenderer_ClearSnapshot`,
`FCE_ObjectRenderer_Clear`, pumped once per frame. Three things learned the hard way:

1. **Call `FCE_ObjectRenderer_Clear()` after every single object.** Each
   `RenderObject` loads that object into the engine and never releases it on its own.
   Without the per-object clear, opening a folder with a few hundred props inflates
   memory until the editor dies. With it, memory held around 700-710 MB through a
   multi-hour run.
2. **Keep a small bounded queue** — one render in flight, a timeout, and a blacklist.
   An unbounded queue turns a single pathological object into dozens of stalled slots.
3. **Do not bake the whole library.** Measured throughput on a live run: about
   **207 objects/hour** (~349 seconds each). Against 30,130 entries that is ~145 hours
   and ~5.6 GB of PNGs, of which ~5.1 GB is the uncontrollable `png_src` dump the
   engine writes for you. Bake per folder, on demand, and treat any offer to bake the
   full library as the wrong goal.

A whole-library pass also has a vendor route — `-generateObjectThumbnails` /
`-resizeThumbnails` on the editor command line, and a long native
`-generateObjectThumbnails` pass — but it is the same arithmetic.

## What is in `thumbnails/`

2,050 PNGs, **128x128, 8-bit RGBA, transparent background**, each named
`<object id>.png` after the numeric id the library entry carries.

These came out of lookup 3 above: rendered by Far Cry 3's own object renderer on the
author's machine, from a library that includes a third-party object pack. They are
provided so you can see what a populated set looks like and diff it against your own.

Drop the folder contents into your `<editor install>\bin\thumbnails` and the browser
reads them first, before it ever tries to render — that is the cheap way to make a
folder legible while you work. If you want them where the engine would have put its
own, they need converting to `.xbt` (next section).

Note the coverage: 2,050 icons against a ~30,000-entry library is a small fraction.
This folder is a sample, not a complete pack, and it is not going to become one.

## The `.xbt` container

`.xbt` is a Dunia resource container. For a 2D texture it is three parts
concatenated, and the whole file is exactly `36 + 128 + w*h` bytes for DXT5 — no
wrapper compression, no padding:

```
offset  size  contents
0       36    TBX header
36      128   DDS block: 4-byte "DDS " magic + the standard 124-byte DDS_HEADER
164     w*h   DXT5 / BC3 block data, (w/4)*(h/4)*16 bytes
```

TBX header, little-endian, from real samples on a retail install:

| Offset | Size | Field | Thumbnails (`object_icons`, `pc` fonts) | Terrain texture (`watercloud_n`) |
|--------|------|-------|------------------------------------------|----------------------------------|
| 0      | 3+1  | magic | `TBX` + `0x00` | `TBX` + `0x00` |
| 4      | 4    | ?     | `0x68` | `0x68` |
| 8      | 4    | payload offset | `36` | `80` |
| 12     | 4    | level count | `1` | `2` |
| 16     | 4    | flags | `01 FF FF FF` | `00 FF FF FF` |
| 20     | 4    | per-file hash | varies | varies |
| 24     | 8    | GUID | `b2 84 42 a8 21 22 86 1b` | `e4 68 85 21 64 2a f8 b1` |
| 32     | 4    | ?     | `0` | `1885434471` |

Two things that will bite you if you assume too much. The field at `+20` is a hash
of something outside the file: CRC32 of the payload, of the header, and of the id
were all tested against four real samples and none matched, so for a loose file the
engine probably does not consult it and writing `0` is the pragmatic call — inside a
bigfile TOC it presumably has meaning. And the GUID at `+24` is **not** universal: it
identifies the resource type. The `ingameeditor` icons and the `graphics/fonts`
textures share one GUID, a tattoo texture in `graphics/__fc3_graphics/multiplayer`
uses another, terrain another. Copy the GUID from a file of the same kind as the one
you are imitating.

The DDS block. Offsets below are from the start of the block, so the classic
`DDS_HEADER` field numbers are all shifted by 4 because the magic is in front:

| Offset | Field | Value |
|--------|-------|-------|
| 0      | magic | `DDS ` |
| 4      | `dwSize` | `124` |
| 8      | `dwFlags` | `0x7C` |
| 12     | `dwHeight` | image height |
| 16     | `dwWidth` | image width |
| 20     | `dwPitchOrLinearSize` | `w*h` for DXT5 |
| 24     | `dwMipMapCount` | `0` |
| 28-71  | `dwReserved1[11]` | zero, except the last word |
| 68     | → | `0x5454564E` = `NVTT`, the encoder's signature |
| 76     | pixel-format `dwSize` | `0` (spec says 32) |
| 80     | pixel-format `dwFlags` | `0` (spec says `FOURCC`) |
| 84     | pixel-format `dwFourCC` | `DXT5` |
| 88     | `dwRGBBitCount` | `0` |
| 108    | `dwCaps` | `0x1000` |
| 112    | `dwCaps2` | `0x1000` (`0x401008` on a terrain texture) |
| 120    | `dwCaps4` | `0` |
| 124    | `dwReserved2` | `0xAAAAAAAA` |

Note how many of those values are *wrong* against the official DDS specification:
`dwFlags` carries a small nonstandard bit set, the pixel-format struct claims a size
of zero and no FOURCC flag, `dwMipMapCount` is `0` rather than `1`, and the reserved
tail is the `0xAAAAAAAA` uninitialised-memory sentinel. None of that stops the engine,
because the engine's loader is not a DDS-conformance checker.

Which is the whole point: **do not synthesise this header.** Copy the 128-byte block
verbatim from a `.xbt` the engine already accepts — `ingameeditor/object_icons.xbt` is
the convenient one — then patch only three uint32s: height at `+12`, width at `+16`,
linear size at `+20`. Every vendor quirk stays byte-identical. If you build the header
from the Microsoft spec instead, you will produce a file that every generic DDS tool
accepts and the game may not.

DXT5/BC3 block layout, 16 bytes per 4x4 texel block:

```
0..1     alpha endpoints a0, a1   (a0 = max alpha, a1 = min alpha)
2..7     48 bits, 3-bit alpha index per texel, texel order row-major
8..9     colour endpoint 0, RGB565
10..11   colour endpoint 1, RGB565
12..15   32 bits, 2-bit colour index per texel
```

If `a0 > a1` the eight alpha samples are evenly spaced between them; otherwise indices
0..5 interpolate and index 6 is `0` and index 7 is `255`. Texels clamp at the right and
bottom edges for non-multiple-of-4 dimensions.

For reference, the `pc` store is not the only icon source in the engine:
`ingameeditor/object_icons.xbt` is a 512x512 DXT5 atlas of about 40 generic editor
glyphs — spawn arrows, zone letters, radio, car, deer. Strip the 36-byte TBX header,
decode the DDS, and you can look at it directly. It is not a prop thumbnail source and
browsing it will not fill your grid.

## Not in this repository (yet)

The encoder that turns a PNG into the `pc\<id>.xbt` shape described above is written and
round-trip checked, but it has not been moved here; neither has the PNG/DDS decoder used
to verify it. The format notes above are complete enough to reimplement from, and are
released under Part A of the license, so that is on purpose. If you want the tools in the
repo, open an issue and say which one.

## Corrections welcome

Everything measurable above was measured on the author's install: Far Cry 3 Steam build,
the decompiled 2012 C# editor, and a third-party object pack. Paths, counts and timings
are from that setup. If yours differs, the difference is the interesting part — please
open an issue with what you found rather than assuming the note is wrong.
