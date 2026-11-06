# Asset Generation

> Generate object-browser thumbnails from a local Far Cry 3 installation.

## 1. Scope

This document describes the measured renderer-driven workflow for producing object thumbnails. It does not publish the PNG encoder, the DXT5 encoder, or the decoder used to verify them.

Function names are observed in the decompiled 2012 C# editor build. Signatures are build-specific; treat the names as a map, not a portable API.

## 2. Constraints

| Constraint | Reason |
|------------|--------|
| Work per folder, on demand | A full-library pass takes about 145 hours and about 5.6 GB. |
| One render in flight | Unbounded queues turn one pathological object into many stalled slots. |
| Call `FCE_ObjectRenderer_Clear()` after every object | `RenderObject` loads the object and never releases it on its own. |
| Timeout every render | Some objects never produce a ready snapshot. |
| Blacklist repeat failures | Retrying a pathological object forever blocks the folder. |
| Use a local, legally purchased installation | The repository grants no rights to game assets. |
| Do not redistribute outputs | The images are Ubisoft-derived. |

## 3. Pipeline

```text
object id
  -> FCE_ObjectRenderer_SetActive
  -> FCE_ObjectRenderer_RenderObject
  -> poll FCE_ObjectRenderer_IsSnapshotReady (with timeout)
  -> FCE_ObjectRenderer_GetSnapshot
  -> persist the snapshot as PNG
  -> FCE_ObjectRenderer_ClearSnapshot
  -> FCE_ObjectRenderer_Clear
```

The renderer is pumped once per frame. Do not spin in a tight blocking loop; let the editor process its frame while the snapshot is pending.

## 4. Renderer functions

| Function | Role |
|----------|------|
| `FCE_ObjectRenderer_SetActive` | Enables the renderer for the current frame. |
| `FCE_ObjectRenderer_RenderObject` | Loads and renders one object. |
| `FCE_ObjectRenderer_IsSnapshotReady` | Reports whether the snapshot is complete. |
| `FCE_ObjectRenderer_GetSnapshot` | Retrieves the rendered image. |
| `FCE_ObjectRenderer_ClearSnapshot` | Releases the snapshot. |
| `FCE_ObjectRenderer_Clear` | Releases the loaded object and renderer state. |

## 5. Queue design

A safe queue has five properties:

1. **One render in flight.** Do not submit the next object until the current one is complete, timed out, or blacklisted.
2. **A timeout.** Treat an object that does not become ready within the timeout as failed.
3. **A blacklist.** Record the ID and the reason. Do not retry it in the same pass.
4. **A checkpoint.** Record completed IDs so a failed batch can resume without repeating work.
5. **A bounded scope.** Queue one folder, not the entire library.

Log the object ID, the state transition, the elapsed time, and the failure reason. A queue that fails silently is worse than a slow queue.

## 6. Memory management

`FCE_ObjectRenderer_RenderObject` loads the object into the engine. It does not release the object when the snapshot is ready.

- Call `FCE_ObjectRenderer_ClearSnapshot()` after retrieving the image.
- Call `FCE_ObjectRenderer_Clear()` after every single object.
- In a measured multi-hour run with per-object clears, memory held around 700–710 MB.
- Without the per-object clear, a folder with a few hundred props inflates memory until the editor dies.
- Stop the batch if memory climbs continuously. Restarting the editor and resuming from a checkpoint is safer than pushing through.

## 7. Throughput and storage

The measured live-run rate was about **207 objects/hour**.

```text
3600 seconds / 207 objects = 17.4 seconds per object
30,130 objects / 207 objects/hour = 145.6 hours
```

| Scope | Objects | Approximate time | Notes |
|-------|---------|------------------|-------|
| Small folder | 100 | 29 minutes | Start here. |
| Large folder | 1,000 | 4 hours 50 minutes | Check memory before expanding. |
| Full library | 30,130 | 145 hours | About 5.6 GB of PNGs. |
| `png_src` share | — | — | About 5.1 GB of the total, written by the engine. |

The full-library storage figure is measured. Per-folder storage scales roughly with the object count; measure your own folder before committing disk space.

The earlier note stated "about 349 seconds per object" alongside the 207 objects/hour rate. That figure is arithmetically inconsistent with both the hourly rate and the 145-hour full-library estimate. The corrected derived figure is about 17.4 seconds per object. See [../CHANGELOG.md](../CHANGELOG.md).

## 8. Output paths

| Path | Contents | Who writes it | Should you edit it? |
|------|----------|---------------|---------------------|
| `<editor install>\bin\thumbnails\<key>.png` | 128x128 normalised PNG cache | The editor build | Yes. This is the cache you control. |
| `<editor data>\ingameeditor\thumbnails\png_src\<id>.png` | Full-resolution, tight-cropped PNG | `FC3.dll`, as a render side effect | No. Expect it to grow. |
| `<editor data>\ingameeditor\thumbnails\pc\<id>.xbt` | Baked DXT5 icons | Vendor builds, or a converter | Yes, only if you can produce a valid `.xbt`. |

## 9. Converting to `.xbt`

The encoder is not published. A reimplementation needs to:

1. Encode the PNG as DXT5/BC3.
2. Copy the 128-byte DDS block from a known-good `.xbt`, such as `ingameeditor/object_icons.xbt`.
3. Patch height at `+12`, width at `+16`, and linear size at `+20`.
4. Build the 36-byte TBX header with the thumbnail payload offset, level count, flags, and resource GUID.
5. Concatenate the header, DDS block, and block data.
6. Verify the result in the editor.

Do not synthesise the DDS header from the Microsoft specification. Several vendor values are nonstandard, and the engine's loader is not a conformance checker. The byte layout is in [TECHNICAL-REFERENCE.md](TECHNICAL-REFERENCE.md).

## 10. Vendor command-line routes

A whole-library pass also has vendor routes:

- `-generateObjectThumbnails` on the editor command line.
- `-resizeThumbnails` on the editor command line.
- A long native `-generateObjectThumbnails` pass.

These routes do not change the arithmetic. They still produce a large `png_src` dump and still take hours. Use them only if you have measured the cost on your own installation.

## 11. Sampling plan

A practical plan:

1. Pick the folder you actually need.
2. Render 10–20 objects.
3. Verify the output path, dimensions, memory, and timing.
4. Expand to the full folder.
5. Checkpoint completed IDs.
6. Stop. Do not continue into the next folder simply because the queue is running.

If coverage is the goal, generate per folder as you work. A complete library is not a useful target at 145 hours for a sample that will still be partial against future object packs.

## 12. Failure modes

| Symptom | Likely cause | Response |
|---------|--------------|----------|
| Memory climbs continuously | Missing per-object `Clear` | Stop, add the clear, restart, resume from checkpoint. |
| Queue stalls on one object | Object never becomes ready | Timeout and blacklist it. |
| Editor dies during a folder | Unbounded queue or missing clears | Reduce concurrency and check memory. |
| Tiles stay blank after generation | Output is in the wrong path | Move the 128x128 PNGs to `bin\thumbnails`. |
| `png_src` grows by gigabytes | Expected engine behaviour | Plan storage; work per folder. |
| Generated `.xbt` is rejected | Bad header or wrong GUID | Copy the DDS block and GUID from the same kind of resource. |
| Throughput is slower than expected | Pathological objects or disk pressure | Check the blacklist and the disk. |

## 13. Checklist

- [ ] The game and editor are installed and working.
- [ ] The installation is backed up.
- [ ] The batch is limited to one folder.
- [ ] One render is in flight.
- [ ] Every render has a timeout.
- [ ] Repeat failures are blacklisted.
- [ ] `FCE_ObjectRenderer_Clear()` runs after every object.
- [ ] Memory is monitored.
- [ ] Completed IDs are checkpointed.
- [ ] Output lands in `<editor install>\bin\thumbnails`.
- [ ] The licensing boundary for generated images is understood.
