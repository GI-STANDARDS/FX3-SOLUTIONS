# Changelog

All notable changes to this project are recorded here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and the project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.1] - 2026-11-06

### Added

- `docs/INSTALLATION.md`, `docs/USAGE.md`, `docs/ASSET-GENERATION.md`, `docs/TECHNICAL-REFERENCE.md`, `docs/FAQ.md`, and `docs/COMPATIBILITY.md`.
- `licenses/LICENSE-DOCUMENTATION.txt` for CC BY-NC 4.0 documentation.
- `licenses/LICENSE-CODE.txt` for PolyForm Noncommercial 1.0.0 code.
- `licenses/LICENSE-THIRD-PARTY.txt` for external ownership boundaries.
- `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `THIRD-PARTY-NOTICES.md`, `CITATION.cff`, and `.gitignore`.
- GitHub issue forms, issue configuration, and a pull request template.

### Changed

- Restructured `README.md` into an overview with links to detailed documentation.
- Restructured `LICENSE` into documentation, code, and third-party sections with canonical notices under `licenses/`.
- Moved the detailed icon-lookup, `.xbt`, TBX, DDS, and DXT5/BC3 reference material into `docs/TECHNICAL-REFERENCE.md`.
- Moved the renderer workflow, memory guidance, and throughput figures into `docs/ASSET-GENERATION.md`.

### Fixed

- Corrected the per-object generation time. A measured rate of 207 objects/hour corresponds to about 17.4 seconds per object; the previous note stated about 349 seconds per object, which is inconsistent with the hourly rate and the ~145-hour full-library estimate. The 145-hour estimate is unchanged.
- Aligned the DDS header field map with the standard `DDS_HEADER` layout: `dwDepth` at `+24`, `dwMipMapCount` at `+28`, and `dwReserved1[11]` at `+32` to `+75`. Observed values are unchanged.
- Clarified that the thumbnail set ships as `thumbnails/thumbnails.zip` (2,050 PNGs), not as an extracted directory tree.

## [0.1.0] - 2026-11-06

### Added

- Initial public documentation of the Far Cry 3 object-browser blank-tile problem.
- Icon lookup order for the baked `pc` `.xbt` store, `png_src`, and the editor's 128x128 cache.
- Coverage measurements for the stock library and third-party object packs.
- Renderer-driven thumbnail generation notes, memory guidance, and throughput figures.
- `.xbt` container, TBX header, DDS block, and DXT5/BC3 layout notes.
- `thumbnails/thumbnails.zip` with 2,050 128x128 PNGs.
- Initial `LICENSE` notice covering original documentation and excluding Ubisoft material.
