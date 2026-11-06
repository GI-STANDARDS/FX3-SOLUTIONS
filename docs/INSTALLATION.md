# Installation

> Prepare a supported Far Cry 3 editor environment and install the sample thumbnail set.

## 1. Scope

This repository has no installer and no executable release. Installation means preparing the game and editor environment, then placing optional files where the editor can read them.

There are three workflows:

| Workflow | Installation required |
|----------|-----------------------|
| Read the documentation | None |
| Use the sample thumbnails | Extract `thumbnails/thumbnails.zip` |
| Generate your own icons | A working editor build and the renderer workflow |

## 2. Requirements

| Requirement | Needed for | Notes |
|-------------|------------|-------|
| A legally purchased Far Cry 3 PC copy | Any asset or render work | The repository grants no rights to the game. |
| The Far Cry 3 in-game editor | Browsing and rendering | The editor is part of the supported PC build. |
| A decompiled build of the 2012 C# editor | Renderer-driven generation | Not distributed here; its own terms apply. |
| Windows | Running the editor | The editor is a Windows application. Linux and Proton are untested. |
| A ZIP archive tool | Extracting the sample thumbnails | Any standard archive tool. |
| Free disk space | Generated PNGs | A full-library pass can produce about 5.6 GB. |
| A backup of maps, saves, and the editor install | Safe experimentation | Modding can corrupt saves and maps. |

### 2.1 Game and editor

The measurements in this repository were taken on a Steam PC build of Far Cry 3 with the decompiled 2012 C# editor. Other storefronts may use the same layout, but that is not verified. See [COMPATIBILITY.md](COMPATIBILITY.md).

The renderer workflow depends on function names observed in that editor build. Function signatures are build-specific. If your build differs, treat the names as a map rather than a drop-in API.

### 2.2 Operating system

The editor is a Windows application. No Linux, Proton, or virtual-machine configuration has been tested. Do not assume that path separators or case sensitivity behave the same outside Windows.

### 2.3 Disk and memory

A full-library thumbnail pass is expensive:

| Resource | Measured or estimated cost |
|----------|----------------------------|
| Time | About 207 objects/hour, or about 17.4 seconds per object |
| Full-library time | About 145 hours for 30,130 entries |
| PNG storage | About 5.6 GB |
| `png_src` share | About 5.1 GB, written by the engine |
| Editor memory | About 700–710 MB during a measured multi-hour run with per-object clears |
| Editor memory without clears | Grows until the editor dies |

Work per folder, not per library. The full arithmetic is in [ASSET-GENERATION.md](ASSET-GENERATION.md).

### 2.4 Backup

Back up maps, saves, and the editor installation before you begin. Far Cry 3 modding can break an installation, corrupt a save or a map, and get an online account flagged. This repository does not promise that any technique is safe, correct, or works on your machine.

## 3. Before you begin

### 3.1 Confirm the basics

1. The game launches.
2. The editor launches.
3. The editor opens a map.
4. You have write access to the editor installation directory.
5. You know which object folder you want to work on.

### 3.2 Know the lookup paths

| Path | Role | Safe to edit? |
|------|------|---------------|
| `<editor data>\ingameeditor\thumbnails\pc` | Baked `.xbt` icon store; absent in a Steam install | Do not add PNGs here. It accepts `.xbt` only. |
| `<editor data>\ingameeditor\thumbnails\png_src` | Engine dump from live renders | Do not edit. Expect it to grow. |
| `<editor install>\bin\thumbnails` | 128x128 PNG cache read before rendering | Yes. This is where the sample archive goes. |

### 3.3 Do not modify game archives

Do not edit, repack, or redistribute `patch.dat`, `patch.fat`, `common.dat`, `common.fat`, `.xbt`, `.dds`, or `.fc3map` files belonging to the game. See [../THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md).

## 4. Install the sample thumbnails

### 4.1 Verify the archive

`thumbnails/thumbnails.zip` should contain 2,050 files. Each file is a PNG named after an object ID, with these properties:

| Property | Value |
|----------|-------|
| Dimensions | 128x128 |
| Colour depth | 8-bit RGBA |
| Background | Transparent |
| Naming | `<object id>.png` |

If the count or format differs, stop and verify the download.

### 4.2 Extract

1. Locate `<editor install>\bin\thumbnails`.
2. Create the directory if it does not exist.
3. Extract the archive contents — not the enclosing folder — into that directory.
4. Confirm that the PNGs sit directly in `bin\thumbnails`.

### 4.3 Verify in the editor

1. Start the editor.
2. Open an object folder containing IDs present in the archive.
3. Confirm that the tiles appear without rendering.
4. If the grid is still blank, confirm the files are in `bin\thumbnails`, not `png_src` and not `pc`.

### 4.4 Roll back

Remove the extracted PNGs from `bin\thumbnails`. The editor will fall back to its other lookup paths on the next start.

### 4.5 Coverage and licensing warnings

- The archive is a sample: 2,050 icons against roughly 30,000 entries. It will not fill the whole browser.
- The images are Ubisoft-derived. Do not redistribute, mirror, bundle, or sell them. See [../THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md).

## 5. Prepare a generation environment

### 5.1 Install the game and editor

Use a legally purchased copy. Confirm that the editor can open a map before adding tooling.

### 5.2 Obtain a compatible editor build

The renderer functions are exposed by the decompiled 2012 C# editor build. That build is not distributed here and is subject to its own terms. Confirm that it can render an object before building a queue around it.

### 5.3 Create working directories

```text
<editor install>\bin\thumbnails
<editor data>\ingameeditor\thumbnails\png_src
```

The first is the cache you control. The second is the engine dump you should not edit.

### 5.4 Plan the first batch

Start with one folder and a small number of objects. Measure the time and the memory before expanding. At the measured rate, 100 objects take about 29 minutes and 1,000 objects take about 4 hours 50 minutes.

## 6. Verify the installation

- Open a folder with IDs present in the sample.
- Compare the tiles to the PNGs.
- Confirm that new files appear in `bin\thumbnails` after a render.
- Watch memory in Task Manager during the first batch.
- If memory climbs continuously, stop and check that `FCE_ObjectRenderer_Clear()` runs after every object.
- If the editor stalls, check the queue timeout and blacklist.

## 7. Rollback and removal

1. Remove the extracted sample PNGs from `bin\thumbnails`.
2. Remove generated cache files, keeping a backup if you want to reuse them.
3. Restore maps or saves from backup if the editor corrupted them.
4. Do not delete game archives or the editor installation.

## 8. Next steps

- [USAGE.md](USAGE.md) — choose a workflow and diagnose blank tiles.
- [ASSET-GENERATION.md](ASSET-GENERATION.md) — run the renderer safely.
- [FAQ.md](FAQ.md) — common problems.
- [COMPATIBILITY.md](COMPATIBILITY.md) — tested and untested environments.
