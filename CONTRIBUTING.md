# Contributing to FX3-SOLUTIONS

> How to report corrections, propose changes, and submit contributions without crossing licensing boundaries.

## 1. Before you start

- Search the existing issues and the [documentation index](README.md#8-documentation) before opening a new one.
- Open an issue before a large change: new format findings, new tooling, a repository reorganisation, or a change to the thumbnail archive.
- Small fixes — typos, broken links, clarified wording — can go straight to a pull request.
- Read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) and [SECURITY.md](SECURITY.md).

## 2. What belongs here

| Contribution | Accepted? | Notes |
|--------------|-----------|-------|
| Corrections to paths, counts, measurements, or timings | Yes | Include the environment and the measurement. |
| Compatibility reports | Yes | State the game build, editor build, object pack, and OS. |
| New format findings | Yes | Include evidence and distinguish measured facts from inferences. |
| Documentation improvements | Yes | Follow the style in section 4. |
| Original tooling | Yes, with an issue first | Include a license header and tests or a round-trip check. |
| Game assets, extracted textures, or archives | No | Ubisoft material is not accepted. |
| Pirated content, cracks, or DRM bypasses | No | Out of scope. |
| Redistribution of third-party object packs | No | Link to the author's distribution instead. |
| Personal data, account identifiers, or private paths | No | Redact before posting. |

## 3. Evidence standard

Technical claims in this repository are measured, not guessed. A useful contribution states:

1. What you measured.
2. Which game build, editor build, object pack, and OS you used.
3. The exact steps or file paths, with personal information removed.
4. The result, including counts, sizes, timings, or error text.
5. What is measured and what is inferred.

If you cannot reproduce a result, say so. A negative result or a differing environment is useful; it is not automatically a correction.

## 4. Documentation style

- Use numbered headings: `## 1. Section`, `### 1.1 Subsection`.
- Write formal, direct, technically accurate English.
- Use fenced code blocks with a language identifier: `text`, `csharp`, `c`, `yaml`, `markdown`.
- Use tables for requirements, compatibility, comparisons, and lookup paths.
- Use ISO dates: `YYYY-MM-DD`.
- Use semantic versions: `MAJOR.MINOR.PATCH`.
- Link to official sources where one exists.
- Distinguish legal, technical, and compatibility warnings.
- Keep the README an overview. Put detail in `docs/`.
- Do not add emojis or decorative badges without a functional reason.

## 5. Licensing of contributions

This repository uses two licenses for original work:

| Contribution type | License |
|-------------------|---------|
| Documentation, format notes, and descriptive text | CC BY-NC 4.0 |
| Source code, scripts, and configuration | PolyForm Noncommercial 1.0.0 |

By submitting a contribution, you confirm that you have the right to license it and that you agree to license your original contribution under the applicable terms above.

Do not contribute third-party material unless you identify its license and confirm it is compatible. If your change includes third-party code, include its license and attribution in the pull request and in [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).

Do not contribute Far Cry 3 assets, extracted files, or redistributed object packs. See [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).

Code files should carry a short header:

```text
Required Notice: Copyright (c) 2026 GI-STANDARDS
Licensed under the PolyForm Noncommercial License 1.0.0.
See licenses/LICENSE-CODE.txt.
```

## 6. Pull request process

1. Fork the repository and create a branch from `main`.
2. Make one focused change.
3. Update [CHANGELOG.md](CHANGELOG.md) for user-visible changes.
4. Fill in the pull request template.
5. Link the related issue, if one exists.
6. Do not commit generated caches, extracted assets, or binaries unless the change is specifically about the sample archive and was agreed in an issue first.
7. Expect review. The maintainer may edit wording for consistency or ask for evidence.

## 7. Commit messages

Use an imperative summary that explains why the change exists. A short prefix such as `docs:`, `fix:`, or `chore:` is welcome.

```text
docs: correct per-object render timing
docs: add compatibility note for another storefront
fix: restore broken link to the technical reference
```

## 8. Review and corrections

Technical claims may be challenged with evidence. That is the point of the project. A correction that includes a reproducible measurement is more useful than a preference.

Pull requests that mix unrelated changes, add Ubisoft material, or ignore the licensing boundary may be closed without merging.
