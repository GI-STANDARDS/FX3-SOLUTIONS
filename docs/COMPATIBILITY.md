# Compatibility

> Tested, reported, and unknown Far Cry 3 editor environments.

## 1. What "supported" means

In this repository, "supported" means documented or measured. It does not mean warranted, guaranteed, or endorsed by Ubisoft. A differing environment may work; report the differences with evidence.

## 2. Tested environments

| Component | Environment | Status | Notes |
|-----------|-------------|--------|-------|
| Game | Far Cry 3, Steam PC build | Tested | The setup used for all measurements. |
| Editor | Decompiled 2012 C# build | Tested | Used for renderer calls and cache behaviour. |
| Object library | Third-party object pack | Tested | About 30,000 entries; added props have no baked icons. |
| Thumbnail cache | 128x128 PNG in `bin\thumbnails` | Tested | Read before rendering. |
| `.xbt` samples | DXT5 2D textures | Tested | Thumbnail and terrain samples. |
| DDS parsing | DXT5/BC3 | Tested | Nonstandard header values documented. |
| Sample archive | 2,050 PNGs | Verified | 128x128, 8-bit RGBA, transparent. |

## 3. Reported but not tested

| Environment | Report | Confidence |
|-------------|--------|------------|
| Console builds | Ship the baked `pc` icon store | Reported |
| Disc builds | Same baked store as console | Reported |
| Other PC storefronts | May use the same directory layout | Unverified |
| Other object packs | May expand the library further | Unverified |

A report is not a measurement. Treat these as leads, not guarantees.

## 4. Unknown and untested

- Linux, Proton, and virtual machines.
- Editor builds other than the decompiled 2012 C# build.
- Localised editor builds with different paths.
- Heavily modded game versions.
- `.xbt` resource types other than the thumbnail, font, tattoo, and terrain samples.
- Non-DXT5 texture compression inside `.xbt`.
- Editor builds that do not expose the `FCE_ObjectRenderer_*` functions.

## 5. Compatibility matrix

| Environment | Read the docs | Use the sample cache | Renderer workflow | Convert to `.xbt` |
|-------------|---------------|----------------------|-------------------|-------------------|
| Far Cry 3, Steam PC | Yes | Yes | Yes | Yes, with a compatible encoder |
| Far Cry 3, console or disc | Yes | Not applicable | Unknown | Unknown |
| Other PC storefront | Yes | Likely, verify paths | Unknown | Unknown |
| Linux or Proton | Yes | Unknown | Unknown | Unknown |

"Likely" means the path layout is expected to match but has not been verified.

## 6. Path compatibility

The documented paths are Windows paths:

```text
<editor data>\ingameeditor\thumbnails\pc
<editor data>\ingameeditor\thumbnails\png_src
<editor install>\bin\thumbnails
```

The exact `<editor data>` and `<editor install>` roots depend on the storefront and installation choices. Confirm them on your own machine. Do not assume that a path from this repository exists verbatim.

## 7. Editor build compatibility

The renderer function names are observed in the decompiled 2012 C# editor build. They may be exported differently in another build. Before building a queue, confirm that the build exposes:

- `FCE_ObjectRenderer_SetActive`
- `FCE_ObjectRenderer_RenderObject`
- `FCE_ObjectRenderer_IsSnapshotReady`
- `FCE_ObjectRenderer_GetSnapshot`
- `FCE_ObjectRenderer_ClearSnapshot`
- `FCE_ObjectRenderer_Clear`

If one is missing, the workflow in [ASSET-GENERATION.md](ASSET-GENERATION.md) does not apply unchanged.

## 8. Reporting compatibility

Include:

1. Game platform, storefront, and build.
2. Editor build and version.
3. Object pack and library size.
4. Windows version.
5. The paths that exist on your machine, with personal information removed.
6. What works, what fails, and the exact error.
7. Counts, sizes, timings, and screenshots where relevant.

Open an issue using the bug report form. See [../CONTRIBUTING.md](../CONTRIBUTING.md).

## 9. Related

- [INSTALLATION.md](INSTALLATION.md)
- [USAGE.md](USAGE.md)
- [FAQ.md](FAQ.md)
- [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md)
