# FAQ

> Common problems, coverage questions, and licensing answers.

## 1. Why are the object browser tiles blank?

The editor resolves icons through a baked `.xbt` store, a session dump, and a 128x128 cache. A Steam installation does not contain the baked store. If nothing has been rendered, the cache is also empty. The grid is blank because all three lookups fail, not because the map or installation is broken.

See [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md).

## 2. Is my map or installation corrupt?

Usually not. A blank grid is the expected result of the missing baked store. Verify that the map opens, that the editor renders objects, and that the three lookup paths are in the state described in the technical reference before reinstalling anything.

## 3. Why do some editors on YouTube show a full grid?

They are usually using the stock library, which has nearly complete icon coverage, or the console build, which ships the baked `pc` store. A heavily expanded library with a Steam install will not look like that without generating icons.

## 4. Can I bake the entire library?

It is technically possible but impractical. The measured rate is about 207 objects/hour. Against 30,130 entries that is about 145 hours and about 5.6 GB of PNGs, of which about 5.1 GB is the engine's `png_src` dump. Bake per folder, on demand.

## 5. How long does generation take?

At the measured rate, about 17.4 seconds per object:

| Objects | Approximate time |
|---------|------------------|
| 100 | 29 minutes |
| 1,000 | 4 hours 50 minutes |
| 30,130 | 145 hours |

Pathological objects and disk pressure can make it slower. See [ASSET-GENERATION.md](ASSET-GENERATION.md).

## 6. Why is `png_src` so large?

`FC3.dll` writes a full-resolution, tight-cropped PNG for every live render. It is an engine dump, not a compact cache. The full-library pass produces about 5.1 GB in that directory alone. Do not edit it; plan for it.

## 7. Where did the sample thumbnails come from?

They were rendered by Far Cry 3's own object renderer on the author's machine, from a library that includes a third-party object pack. They are the editor's 128x128 cache output, archived as `thumbnails/thumbnails.zip`.

## 8. Are the sample thumbnails licensed?

No. They are Ubisoft-derived material. The author claims no copyright in them and grants no license to them. The archive is provided for private study, personal backup, and NonCommercial modding on top of your own copy of the game. See [../THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md).

## 9. Can I redistribute the sample thumbnails?

Do not. The repository does not grant redistribution rights, and the author is not certain that republishing engine-rendered game assets is permitted by Ubisoft's end-user license. If you want a set, generate your own from your own installation.

## 10. Can I use the sample thumbnails commercially?

No. They are not licensed for commercial use, and the underlying props belong to Ubisoft.

## 11. Why only 2,050 icons?

The archive is a sample, not a complete pack. The expanded library has about 30,000 entries, so the sample covers a small fraction. It exists to show the shape of a populated set and to make a folder legible while you work.

## 12. Can I put PNGs in the `pc` folder?

No. The `pc` store accepts `.xbt` only. There is no `pc\*.png` lookup path. PNG caches belong in `<editor install>\bin\thumbnails`.

## 13. Can I generate `.xbt` files?

The format is documented, but the encoder is not published. You can reimplement it: encode DXT5/BC3, copy the DDS block from a known-good `.xbt`, patch height, width, and linear size, and prepend the TBX header. See [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md).

## 14. Where is the encoder or decoder?

It is written and round-trip checked but has not been moved into this repository. Open an issue and say which tool you need. The documentation license permits you to reimplement from the format notes.

## 15. Which game and editor versions are supported?

The measurements come from a Far Cry 3 Steam PC build and the decompiled 2012 C# editor. Other storefronts, editor builds, and game versions are untested. See [COMPATIBILITY.md](COMPATIBILITY.md).

## 16. The editor crashes or memory grows. What do I do?

Stop the batch. Check that `FCE_ObjectRenderer_Clear()` runs after every object, that the queue has one render in flight, and that every render has a timeout and a blacklist. Memory held around 700–710 MB in a measured run with per-object clears. Without them, it grows until the editor dies. See [ASSET-GENERATION.md](ASSET-GENERATION.md).

## 17. Does this work on console or disc builds?

Those builds are reported to ship the baked `pc` icon store, so the blank-grid problem may not occur. They have not been measured here. Do not assume that file paths or editor behaviour match the Steam build.

## 18. Does this work on Linux or Proton?

Untested. The editor is a Windows application, and the path layouts in this repository are Windows paths. If you test it, report the environment and the differences.

## 19. How do I report a correction?

Open an issue with the environment, the steps, and the measurement. Separate what you measured from what you inferred. Do not attach game files or personal data. See [../CONTRIBUTING.md](../CONTRIBUTING.md).

## 20. How do I cite the project?

Use [../CITATION.cff](../CITATION.cff). The original documentation is CC BY-NC 4.0; the game material is excluded.
