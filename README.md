# FX3-SOLUTIONS

> Technical documentation for Far Cry 3 sandbox and map-editor thumbnail handling.

FX3-SOLUTIONS documents problems in the 2012 Far Cry 3 editor that were never covered by vendor documentation: why the object browser shows blank tiles, how the editor resolves thumbnail paths, how to generate icons from a local installation, and what the `.xbt` texture container actually contains.

The project is documentation-first. It does not distribute the game, the editor, the encoder, or any tool that bypasses licensing. The `thumbnails/thumbnails.zip` archive is a sample of engine-rendered, Ubisoft-derived imagery provided for comparison and private study. It is not licensed for redistribution.

**Documentation release:** 0.1.1 (2026-11-06)  
**Repository:** `https://github.com/GI-STANDARDS/FX3-SOLUTIONS`  
**Status:** no original tooling is published yet.

---

## 1. Overview

### 1.1 The problem

You open the Far Cry 3 in-game editor, browse an object folder, and the grid is almost entirely blank. Not broken — blank. You hover a slot and it either paints nothing or paints the same generic glyph.

The map is not corrupt, the installation is not missing textures, and the `.fc3map` is not at fault. The editor's icon lookup is failing because the baked icon store is absent, which is the normal state of a Steam installation.

### 1.2 The finding

The editor resolves an icon through three paths, in this order:

| # | Path | Who writes it | What it holds |
|---|------|---------------|---------------|
| 1 | `<editor data>\ingameeditor\thumbnails\pc\<id>.xbt` | Nobody, in a Steam install | Baked DXT5 icons — the store the vendor shipped on console and disc builds |
| 2 | `<editor data>\ingameeditor\thumbnails\png_src\<id>.png` | `FC3.dll`, as a side effect of a live render | Full-resolution, tight-cropped PNG of whatever was just rendered |
| 3 | `<editor install>\bin\thumbnails\<key>.png` | The decompiled editor build, at runtime | 128x128 normalised PNG cache |

Lookup 1 is the intended path. `FC3.dll` carries the format string `ingameeditor\thumbnails\pc\%u%s.xbt` — plus `pc\col%u.xbt`, `pc\wat%u.xbt`, and a `console\` mirror — so every icon in the game normally arrives as a pre-baked `.xbt`. A Steam installation does not contain that folder. There is no `pc\*.png` variant either: the `pc` store accepts `.xbt` only.

Lookup 2 contains only what you have already rendered in the current session. Lookup 3 contains only what the editor managed to render before you stopped looking. With no baked store and nothing cached, the browser is blank until something renders.

The full lookup analysis, including the exact format strings and the `.xbt` layout, is in [docs/TECHNICAL-REFERENCE.md](docs/TECHNICAL-REFERENCE.md).

### 1.3 Why coverage collapsed

Ubisoft's icon set was built for the stock library. Measured against the shipped `ingameeditor` data: roughly **2,318 baked icons for roughly 2,336 stock objects** — a complete set for the game as released.

Popular object packs expand the library to about **30,000 entries**, roughly **13x the prop count with 1x the icon coverage**. Every added prop has no baked icon and no cache entry, so it is blank by definition. Editors on YouTube that appear "full of icons" are either using the stock library or the console build of the editor, which ships the `pc` store. This is worth knowing before spending a day chasing a bug that is not there.

### 1.4 What this repository provides

- A diagnosis of the blank object browser and the three-path icon lookup.
- A measured, per-folder workflow for generating icons without exhausting memory or disk.
- A 2,050-image sample of rendered thumbnails for comparison.
- The `.xbt`, TBX, DDS, and DXT5/BC3 layouts needed to reimplement PNG-to-`.xbt` conversion.
- Compatibility notes and a measurement record that distinguishes measured facts from inferences.

No separate screenshots are published. The sample archive is the visual reference.

---

## 2. Requirements

| Requirement | Needed for | Notes |
|-------------|------------|-------|
| A legally purchased Far Cry 3 PC copy | Any asset or render work | This repository grants no rights to the game. |
| The Far Cry 3 in-game editor | Browsing and rendering | The editor is part of the supported PC build. |
| A decompiled build of the 2012 C# editor | Renderer-driven generation | Not distributed here; its own terms apply. |
| Windows | Running the editor | The editor is a Windows application. Linux and Proton are untested. |
| A ZIP archive tool | Extracting the sample thumbnails | Any standard archive tool. |
| Free disk space | Generated PNGs | A full-library pass can produce about 5.6 GB. See [docs/ASSET-GENERATION.md](docs/ASSET-GENERATION.md). |
| A backup of maps, saves, and the editor install | Safe experimentation | Modding can corrupt saves and maps. |

Do not proceed without a backup. Far Cry 3 modding can break an installation, corrupt a save or a map, and get an online account flagged. Nothing here promises that any technique is safe, correct, or works on your machine.

---

## 3. Installation

No installation is required to read the documentation. Two optional workflows require files to be placed on disk.

### 3.1 Use the sample thumbnails

1. Download `thumbnails/thumbnails.zip` from this repository.
2. Extract the archive contents into `<editor install>\bin\thumbnails`.
3. Start the editor and open an object folder.

The editor reads lookup 3 before it attempts to render, so the extracted PNGs make the folder legible immediately. To roll back, remove the extracted PNGs. The archive contains 2,050 icons against a library of roughly 30,000 entries; it is a sample, not a complete pack.

### 3.2 Prepare a generation environment

The short version: install the game and editor, obtain a working decompiled editor build, confirm it opens a map, and create the cache directories. The full procedure, including verification and rollback, is in [docs/INSTALLATION.md](docs/INSTALLATION.md).

### 3.3 Verify the setup

Open a folder containing objects whose IDs are present in the sample, compare the tiles against the archive, and confirm that new PNGs appear in `<editor install>\bin\thumbnails` after a render. Watch memory during the first batch; it should stay near the measured 700–710 MB range rather than climb continuously.

---

## 4. Usage

| Task | Start here |
|------|------------|
| Understand why the browser is blank | [docs/TECHNICAL-REFERENCE.md](docs/TECHNICAL-REFERENCE.md) |
| Make an existing folder legible | [docs/USAGE.md](docs/USAGE.md) |
| Generate icons for one folder | [docs/ASSET-GENERATION.md](docs/ASSET-GENERATION.md) |
| Convert PNGs to `.xbt` | [docs/TECHNICAL-REFERENCE.md](docs/TECHNICAL-REFERENCE.md) |
| Diagnose a problem | [docs/FAQ.md](docs/FAQ.md) |

### 4.1 Diagnose blank tiles

Check the three lookup paths. In a Steam installation, the `pc` store is absent, `png_src` contains only session renders, and `bin\thumbnails` contains only prior cache entries. A blank grid is the expected result, not evidence of a broken map or install.

### 4.2 Make a folder legible

Extract the sample archive into `<editor install>\bin\thumbnails`, or generate icons for the folder on demand. Do not copy PNG files into the `pc` store; that store accepts `.xbt` only.

### 4.3 Generate icons on demand

The live renderer is the practical route: `FCE_ObjectRenderer_SetActive`, `FCE_ObjectRenderer_RenderObject`, `FCE_ObjectRenderer_IsSnapshotReady`, `FCE_ObjectRenderer_GetSnapshot`, `FCE_ObjectRenderer_ClearSnapshot`, `FCE_ObjectRenderer_Clear`, pumped once per frame.

Three rules matter:

1. **Call `FCE_ObjectRenderer_Clear()` after every object.** `RenderObject` loads the object into the engine and never releases it on its own. Without the per-object clear, opening a folder with a few hundred props inflates memory until the editor dies. With it, memory held around 700–710 MB through a multi-hour run.
2. **Keep a small bounded queue.** One render in flight, a timeout, and a blacklist. An unbounded queue turns a single pathological object into dozens of stalled slots.
3. **Do not bake the whole library.** Measured throughput on a live run was about **207 objects/hour** (about **17.4 seconds per object**). Against 30,130 entries that is about **145 hours** and roughly **5.6 GB of PNGs**, of which about **5.1 GB** is the uncontrollable `png_src` dump the engine writes for you. Bake per folder, on demand, and treat any offer to bake the full library as the wrong goal.

A whole-library pass also has a vendor route — `-generateObjectThumbnails` and `-resizeThumbnails` on the editor command line, and a long native `-generateObjectThumbnails` pass — but it is the same arithmetic. The detailed workflow is in [docs/ASSET-GENERATION.md](docs/ASSET-GENERATION.md).

### 4.4 Convert PNGs to `.xbt`

The encoder is not published yet. The format notes are complete enough to reimplement from: encode DXT5/BC3, copy the 128-byte DDS block from a known-good `.xbt`, patch only height, width, and linear size, and prepend the 36-byte TBX header. Do not synthesise the DDS header from the Microsoft specification; the engine's loader is not a conformance checker. See [docs/TECHNICAL-REFERENCE.md](docs/TECHNICAL-REFERENCE.md).

---

## 5. Compatibility

| Environment | Status | Notes |
|-------------|--------|-------|
| Far Cry 3, Steam PC build | Measured | The setup used for all measurements in this repository. |
| Far Cry 3, console and disc builds | Reported, not measured | These builds ship the baked `pc` icon store. |
| Decompiled 2012 C# editor | Measured | Used for the renderer calls and cache behaviour. |
| Third-party object packs | Measured | About 30,000 entries; no baked icons for added props. |
| Other PC storefronts | Unknown | Paths may match; verify against the lookup table. |
| Linux and Proton | Unknown | The editor is a Windows application; untested. |
| Other `.xbt` resource types | Partially measured | The TBX GUID varies by resource type. |

"Supported" here means documented or measured, not warranted. Report a differing environment with evidence. See [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md).

---

## 6. Troubleshooting

| Symptom | Likely cause | Action |
|---------|--------------|--------|
| Every tile is blank | No baked `pc` store and no cache | Use the sample archive or render the folder. |
| Only some tiles appear | The cache is partial | Render the remaining objects per folder. |
| PNGs exist but the browser stays blank | Files are in the wrong lookup path | Move 128x128 PNGs to `<editor install>\bin\thumbnails`. |
| Memory rises until the editor dies | `FCE_ObjectRenderer_Clear()` is not called per object | Add the per-object clear and restart. |
| The queue stalls on one prop | A pathological object blocks the queue | Add a timeout and blacklist that object. |
| `png_src` consumes gigabytes | The engine writes a full-resolution dump per render | Work per folder and plan storage. |
| A generated `.xbt` is rejected | Header was synthesised or copied from the wrong resource type | Copy the 128-byte DDS block and the GUID from the same kind of file. |

More answers are in [docs/FAQ.md](docs/FAQ.md).

---

## 7. Repository layout

```text
FX3-SOLUTIONS/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.yml
│   │   ├── feature_request.yml
│   │   └── config.yml
│   └── PULL_REQUEST_TEMPLATE.md
├── docs/
│   ├── ASSET-GENERATION.md
│   ├── COMPATIBILITY.md
│   ├── FAQ.md
│   ├── INSTALLATION.md
│   ├── TECHNICAL-REFERENCE.md
│   └── USAGE.md
├── licenses/
│   ├── LICENSE-CODE.txt
│   ├── LICENSE-DOCUMENTATION.txt
│   ├── LICENSE-THIRD-PARTY.txt
│   └── README.md
├── thumbnails/
│   └── thumbnails.zip
├── .gitignore
├── CHANGELOG.md
├── CITATION.cff
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
└── THIRD-PARTY-NOTICES.md
```

`src/` and `scripts/` are planned for original tooling but are not published yet. No original source code is currently in the repository.

---

## 8. Documentation

| Document | Purpose |
|----------|---------|
| [docs/INSTALLATION.md](docs/INSTALLATION.md) | Requirements, sample installation, generation setup, verification, rollback |
| [docs/USAGE.md](docs/USAGE.md) | Workflow selection, diagnosis, per-folder generation, conversion |
| [docs/ASSET-GENERATION.md](docs/ASSET-GENERATION.md) | Renderer pipeline, queue design, memory, throughput, storage |
| [docs/TECHNICAL-REFERENCE.md](docs/TECHNICAL-REFERENCE.md) | Icon lookup, `.xbt`, TBX, DDS, DXT5/BC3 layouts |
| [docs/FAQ.md](docs/FAQ.md) | Common problems and licensing questions |
| [docs/COMPATIBILITY.md](docs/COMPATIBILITY.md) | Tested, reported, and unknown environments |

---

## 9. Licensing

Three kinds of material are covered by three different notices. Do not treat the repository as a single license.

| Material | Terms | Canonical notice |
|----------|-------|------------------|
| Original documentation, format notes, and findings | CC BY-NC 4.0 | [licenses/LICENSE-DOCUMENTATION.txt](licenses/LICENSE-DOCUMENTATION.txt) |
| Original source code, when published | PolyForm Noncommercial 1.0.0 | [licenses/LICENSE-CODE.txt](licenses/LICENSE-CODE.txt) |
| Far Cry 3 and Ubisoft material, including the sample archive | Not licensed here | [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md) |
| Third-party tools, editor source, and object packs | Their own terms | [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md) |

The short version: the text and format notes are reusable non-commercially with attribution; the game assets are not, and the `thumbnails/thumbnails.zip` archive is Ubisoft-derived imagery that you should treat as reference material, not as content you have rights to. Read [LICENSE](LICENSE) and [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md) before republishing anything.

---

## 10. Contributing and security

Corrections, compatibility reports, and format findings are welcome. Read [CONTRIBUTING.md](CONTRIBUTING.md) before opening a pull request. Do not attach game files, extracted assets, account identifiers, or personal paths to an issue.

Report vulnerabilities privately according to [SECURITY.md](SECURITY.md). Community expectations are in [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md).

---

## 11. Corrections

Everything measurable in this repository was measured on the author's installation: a Far Cry 3 Steam build, the decompiled 2012 C# editor, and a third-party object pack. Paths, counts, and timings come from that setup. If yours differs, the difference is the interesting part. Open an issue with what you found rather than assuming the note is wrong.

If you use or reference this work, see [CITATION.cff](CITATION.cff). Version history is in [CHANGELOG.md](CHANGELOG.md).
