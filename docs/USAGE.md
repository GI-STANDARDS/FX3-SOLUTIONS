# Usage

> Choose the right workflow, use the sample set, and generate icons without exhausting memory or disk.

## 1. Choose a workflow

| Goal | Workflow | Start here |
|------|----------|------------|
| Understand why the browser is blank | Read the lookup analysis | [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md) |
| Make an existing folder legible quickly | Extract the sample archive | Section 3 |
| Generate icons for a folder | Renderer workflow | Section 4 and [ASSET-GENERATION.md](ASSET-GENERATION.md) |
| Convert PNGs to `.xbt` | Reimplement the container format | Section 6 and [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md) |
| Diagnose a failure | Check the symptom table | [FAQ.md](FAQ.md) |

## 2. Diagnose blank object tiles

A blank grid is usually not a broken map, a broken install, or missing textures. It is the expected result when the baked icon store is absent and nothing has been rendered yet.

1. Confirm that the grid is blank rather than showing generic editor glyphs.
2. Check `<editor data>\ingameeditor\thumbnails\pc`. In a Steam installation, this directory does not exist. Lookup 1 cannot succeed.
3. Check `<editor data>\ingameeditor\thumbnails\png_src`. It contains only objects rendered in the current session.
4. Check `<editor install>\bin\thumbnails`. It contains only objects the editor already rendered.
5. Conclude that the browser needs either the sample cache or a render pass.

Do not reinstall the game to fix a blank grid. Do not edit the map. The lookup order and the exact format strings are in [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md).

## 3. Use the sample thumbnails

1. Download `thumbnails/thumbnails.zip`.
2. Extract the contents into `<editor install>\bin\thumbnails`.
3. Start the editor and open an object folder.
4. Remove the PNGs to roll back.

The archive contains 2,050 PNGs, each 128x128, 8-bit RGBA, with a transparent background and a filename matching the object ID. The editor reads this cache before it renders, so the folder becomes legible without a render pass.

Coverage warning: 2,050 icons against a roughly 30,000-entry library is a small fraction. The archive is a sample, not a complete pack.

Licensing warning: the images are Ubisoft-derived. Do not redistribute, mirror, bundle, or sell them. See [../THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md).

## 4. Generate icons for one folder

The renderer workflow is documented in detail in [ASSET-GENERATION.md](ASSET-GENERATION.md). The operating rules are:

1. Select one object folder.
2. Enumerate the object IDs you want.
3. Queue one render at a time.
4. Pump the renderer once per frame.
5. Poll `FCE_ObjectRenderer_IsSnapshotReady` with a timeout.
6. Retrieve the image with `FCE_ObjectRenderer_GetSnapshot`.
7. Release the snapshot with `FCE_ObjectRenderer_ClearSnapshot`.
8. Call `FCE_ObjectRenderer_Clear()` after every object.
9. Blacklist a pathological object instead of retrying it forever.
10. Stop when the folder is complete and verify memory.

Do not start with the whole library. At the measured rate, a full pass is about 145 hours and about 5.6 GB of PNGs.

## 5. Understand coverage

| Library | Entries | Baked icons | Approximate coverage |
|---------|---------|-------------|----------------------|
| Stock game | About 2,336 | About 2,318 | Nearly complete |
| Third-party object pack | About 30,000 | About 2,318 | About 8% |
| Sample archive | Not a library | 2,050 | Sample only |

The library grew about 13x while the baked icon count stayed at 1x. New props are blank by definition because they have no baked icon and no cache entry.

Editors that appear full of icons are usually using the stock library or the console build, which ships the `pc` store.

## 6. Convert PNGs to `.xbt`

The encoder is not published. The format notes are sufficient to reimplement it:

1. Encode the image as DXT5/BC3.
2. Copy the 128-byte DDS block from a `.xbt` the engine already accepts — `ingameeditor/object_icons.xbt` is the convenient source.
3. Patch only three uint32 values: height at block offset `+12`, width at `+16`, linear size at `+20`.
4. Build the 36-byte TBX header with payload offset `36`, level count `1`, the flags observed on thumbnails, the GUID from the same kind of resource, and `0` for the per-file hash on a loose file.
5. Concatenate the TBX header, the DDS block, and the DXT5 block data.
6. Place the result in the `pc` store as `<id>.xbt`.
7. Verify it in the editor.

Do not synthesise the DDS header from the Microsoft specification. The engine's loader accepts several nonstandard values, and a spec-perfect header is not guaranteed to work. The byte tables are in [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md).

## 7. Do not

- Bake the whole library as a first step.
- Run an unbounded render queue.
- Skip `FCE_ObjectRenderer_Clear()`.
- Copy PNGs into the `pc` store.
- Synthesise the DDS header.
- Redistribute the sample images.
- Commit game archives or extracted assets to the repository.

## 8. Verification checklist

- [ ] The batch is limited to one folder.
- [ ] One render is in flight at a time.
- [ ] `FCE_ObjectRenderer_Clear()` runs after every object.
- [ ] A timeout and blacklist are configured.
- [ ] Memory stays near the measured 700–710 MB range.
- [ ] Output lands in `<editor install>\bin\thumbnails`.
- [ ] The sample archive's licensing boundary is understood.
