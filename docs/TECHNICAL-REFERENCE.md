# Technical Reference

> Icon lookup, thumbnail formats, the `.xbt` container, and the DXT5/BC3 block layout.

## 1. Scope and evidence

The values in this document were measured on a Far Cry 3 Steam installation, the decompiled 2012 C# editor, and a third-party object pack. Paths, counts, and byte values come from that setup.

Labels in this document:

- **Measured** means observed on that installation.
- **Inferred** means deduced from the measurements but not independently confirmed.
- **Unknown** means the field or behaviour is not identified.

If your installation differs, the difference is useful. Open an issue with the evidence. See [../CONTRIBUTING.md](../CONTRIBUTING.md).

## 2. Icon lookup order

The editor resolves an icon through three paths, in this order:

| # | Path | Who writes it | What it holds |
|---|------|---------------|---------------|
| 1 | `<editor data>\ingameeditor\thumbnails\pc\<id>.xbt` | Nobody, in a Steam install | Baked DXT5 icons — the store the vendor shipped on console and disc builds |
| 2 | `<editor data>\ingameeditor\thumbnails\png_src\<id>.png` | `FC3.dll`, as a side effect of a live render | Full-resolution, tight-cropped PNG of whatever was just rendered |
| 3 | `<editor install>\bin\thumbnails\<key>.png` | The decompiled editor build, at runtime | 128x128 normalised PNG cache |

### 2.1 Lookup 1: the baked `.xbt` store

`FC3.dll` carries the format string:

```text
ingameeditor\thumbnails\pc\%u%s.xbt
```

It also carries variants:

```text
pc\col%u.xbt
pc\wat%u.xbt
console\<same set>
```

Every icon in the game normally arrives as a pre-baked `.xbt`. In a Steam installation, the `pc` directory does not exist.

There is no `pc\*.png` variant of the format string. The `pc` store accepts `.xbt` only. Copying PNGs into it does not satisfy lookup 1.

### 2.2 Lookup 2: `png_src`

`FC3.dll` writes a full-resolution, tight-cropped PNG into:

```text
<editor data>\ingameeditor\thumbnails\png_src\<id>.png
```

This is a side effect of any live render. It contains only objects rendered in the current session. It is an engine dump, not a cache you should curate.

### 2.3 Lookup 3: the editor cache

The decompiled editor build writes a 128x128 normalised PNG cache into:

```text
<editor install>\bin\thumbnails\<key>.png
```

It contains only objects the editor has already rendered. This is the path you can populate directly with the sample archive.

### 2.4 Failure mode

With no baked `pc` store and no cache, the browser is blank, and it stays blank until something renders. A blank grid is not evidence of a corrupt map, a broken install, or missing textures.

## 3. Coverage

Ubisoft's icon set was built for the stock library.

| Library | Entries | Baked icons | Approximate coverage |
|---------|---------|-------------|----------------------|
| Stock game | About 2,336 | About 2,318 | Nearly complete |
| Third-party object pack | About 30,000 | About 2,318 | About 8% |
| Sample archive | Not a library | 2,050 | Sample only |

The library expands about **13x** while the baked icon count stays at **1x**. Every added prop has no baked icon and no cache entry, so it is blank by definition.

Editors that appear full of icons are usually using the stock library or the console build of the editor, which ships the `pc` store.

## 4. Thumbnail sample

`thumbnails/thumbnails.zip` contains 2,050 files.

| Property | Value |
|----------|-------|
| Format | PNG |
| Dimensions | 128x128 |
| Colour depth | 8-bit RGBA |
| Background | Transparent |
| Naming | `<object id>.png` |
| Source | Rendered by Far Cry 3's own object renderer |
| Library | Includes a third-party object pack |

The archive came from lookup 3. It is a sample, not a complete pack, and it will not become one. It is Ubisoft-derived material and is not licensed for redistribution. See [../THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md).

## 5. The `.xbt` container

### 5.1 Overall layout

`.xbt` is a Dunia resource container. For a 2D texture it is three parts concatenated, with no wrapper compression and no padding:

```text
offset  size  contents
0       36    TBX header
36      128   DDS block: 4-byte "DDS " magic + the standard 124-byte DDS_HEADER
164     w*h   DXT5 / BC3 block data, (w/4)*(h/4)*16 bytes
```

Measured thumbnail files are exactly `36 + 128 + w*h` bytes. In general, DXT5 data occupies `ceil(w/4) * ceil(h/4) * 16` bytes; for dimensions that are multiples of 4 this equals `w*h`.

### 5.2 TBX header

The TBX header is little-endian. These values come from real samples on a retail installation:

| Offset | Size | Field | Thumbnails (`object_icons`, `pc` fonts) | Terrain texture (`watercloud_n`) |
|--------|------|-------|------------------------------------------|----------------------------------|
| 0 | 3+1 | magic | `TBX` + `0x00` | `TBX` + `0x00` |
| 4 | 4 | Unknown | `0x68` | `0x68` |
| 8 | 4 | Payload offset | `36` | `80` |
| 12 | 4 | Level count | `1` | `2` |
| 16 | 4 | Flags | `01 FF FF FF` | `00 FF FF FF` |
| 20 | 4 | Per-file hash | Varies | Varies |
| 24 | 8 | GUID | `b2 84 42 a8 21 22 86 1b` | `e4 68 85 21 64 2a f8 b1` |
| 32 | 4 | Unknown | `0` | `1885434471` |

### 5.3 The hash field at `+20`

The field at `+20` is a hash of something outside the file. CRC32 of the payload, of the header, and of the ID were each tested against four real samples; none matched.

For a loose file, the engine probably does not consult the field, and writing `0` is the pragmatic choice. Inside a bigfile table of contents, it presumably has meaning. This is inferred, not confirmed.

### 5.4 The GUID at `+24`

The GUID identifies the resource type. It is not universal:

| Resource | GUID |
|----------|------|
| `ingameeditor` icons and `graphics/fonts` textures | `b2 84 42 a8 21 22 86 1b` |
| A tattoo texture in `graphics/__fc3_graphics/multiplayer` | A different GUID |
| Terrain textures | A different GUID again |

Copy the GUID from a file of the same kind as the one you are imitating. Do not reuse the thumbnail GUID for a different resource type.

### 5.5 DDS block

Offsets below are from the start of the DDS block, so the classic `DDS_HEADER` field numbers are shifted by 4 because the magic is in front.

| Offset | Field | Value |
|--------|-------|-------|
| 0 | magic | `DDS ` |
| 4 | `dwSize` | `124` |
| 8 | `dwFlags` | `0x7C` |
| 12 | `dwHeight` | Image height |
| 16 | `dwWidth` | Image width |
| 20 | `dwPitchOrLinearSize` | `w*h` for DXT5 with multiple-of-4 dimensions |
| 24 | `dwDepth` | `0` |
| 28 | `dwMipMapCount` | `0` |
| 32–75 | `dwReserved1[11]` | Zero, except the `NVTT` word at `+68` |
| 68 | → | `0x5454564E` = `NVTT`, the encoder's signature |
| 76 | Pixel-format `dwSize` | `0` (the specification says `32`) |
| 80 | Pixel-format `dwFlags` | `0` (the specification says `FOURCC`) |
| 84 | Pixel-format `dwFourCC` | `DXT5` |
| 88 | `dwRGBBitCount` | `0` |
| 108 | `dwCaps` | `0x1000` |
| 112 | `dwCaps2` | `0x1000` (`0x401008` on a terrain texture) |
| 120 | `dwCaps4` | `0` |
| 124 | `dwReserved2` | `0xAAAAAAAA` |

The standard `DDS_HEADER` places `dwDepth` at `+24`, `dwMipMapCount` at `+28`, and the 11-word reserved array at `+32` to `+75`. The original measurement note grouped offsets `28–71` under `dwReserved1`; the standard field map is used here so that every byte is accounted for. The observed values are unchanged.

### 5.6 Nonstandard values

Several values are wrong against the official DDS specification:

- `dwFlags` carries a small nonstandard bit set (`0x7C`).
- The pixel-format struct claims a size of zero and no `FOURCC` flag.
- `dwMipMapCount` is `0` rather than `1`.
- The reserved tail is the `0xAAAAAAAA` uninitialised-memory sentinel.

None of this stops the engine, because the engine's loader is not a DDS-conformance checker.

### 5.7 Header rule

**Do not synthesise this header.** Copy the 128-byte block verbatim from a `.xbt` the engine already accepts — `ingameeditor/object_icons.xbt` is the convenient source — then patch only three uint32 values:

| Offset | Field | Value |
|--------|-------|-------|
| `+12` | Height | Target height |
| `+16` | Width | Target width |
| `+20` | Linear size | `w*h` for DXT5 |

Every vendor quirk then stays byte-identical. If you build the header from the Microsoft specification instead, you will produce a file that every generic DDS tool accepts and the game may not.

### 5.8 DXT5/BC3 block layout

DXT5/BC3 uses 16 bytes per 4x4 texel block:

```text
0..1     alpha endpoints a0, a1   (a0 = max alpha, a1 = min alpha)
2..7     48 bits, 3-bit alpha index per texel, texel order row-major
8..9     colour endpoint 0, RGB565
10..11   colour endpoint 1, RGB565
12..15   32 bits, 2-bit colour index per texel
```

If `a0 > a1`, the eight alpha samples are evenly spaced between them. Otherwise, indices 0..5 interpolate, index 6 is `0`, and index 7 is `255`. Texels clamp at the right and bottom edges for non-multiple-of-4 dimensions.

### 5.9 `object_icons.xbt`

The `pc` store is not the only icon source in the engine. `ingameeditor/object_icons.xbt` is a 512x512 DXT5 atlas of about 40 generic editor glyphs — spawn arrows, zone letters, radio, car, deer.

Strip the 36-byte TBX header, decode the DDS, and you can look at it directly. It is not a prop thumbnail source, and browsing it will not fill your grid.

## 6. Reimplementation notes

The encoder that turns a PNG into the `pc\<id>.xbt` shape is written and round-trip checked, but it has not been moved into this repository. The PNG/DDS decoder used to verify it has not been moved either.

The format notes above are complete enough to reimplement from. That is deliberate: they are released under the documentation license in [../licenses/LICENSE-DOCUMENTATION.txt](../licenses/LICENSE-DOCUMENTATION.txt). If you want the tools in the repository, open an issue and say which one.

When you implement a converter:

1. Use a DXT5/BC3 encoder you have the right to use.
2. Copy the DDS block from a known-good `.xbt`.
3. Patch only height, width, and linear size.
4. Reuse the TBX GUID for the same resource kind.
5. Write `0` for the per-file hash on a loose file.
6. Verify the output in the editor, not only in a generic DDS viewer.

## 7. Measurement record

| Item | Value | Confidence |
|------|-------|------------|
| Game | Far Cry 3, Steam PC build | Measured |
| Editor | Decompiled 2012 C# build | Measured |
| Object pack | Third-party pack included | Measured |
| Stock icons | About 2,318 | Measured |
| Stock objects | About 2,336 | Measured |
| Expanded library | About 30,000 entries | Measured |
| Sample archive | 2,050 PNGs | Measured |
| Per-object generation time | About 17.4 seconds | Derived from 207 objects/hour |
| Full-library time | About 145 hours for 30,130 entries | Derived |
| Full-library PNG storage | About 5.6 GB | Measured |
| `png_src` share | About 5.1 GB | Measured |
| Editor memory | About 700–710 MB with per-object clears | Measured |
| TBX hash field | Unknown algorithm | Tested, unmatched |

## 8. References

- Microsoft, DirectDraw Surface programming guide: `https://learn.microsoft.com/en-us/windows/win32/direct3ddds/dx-graphics-dds-pguide`
- Creative Commons, CC BY-NC 4.0: `https://creativecommons.org/licenses/by-nc/4.0/`
- PolyForm Project, PolyForm Noncommercial 1.0.0: `https://polyformproject.org/licenses/noncommercial/1.0.0`
- Repository changelog: [../CHANGELOG.md](../CHANGELOG.md)
- Third-party notices: [../THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md)
