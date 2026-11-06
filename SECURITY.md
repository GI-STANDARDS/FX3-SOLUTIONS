# Security Policy

> How to report vulnerabilities in this repository and what is in scope.

## 1. Supported versions

| Version | Supported | Notes |
|---------|-----------|-------|
| 0.1.x | Yes | Current documentation line. |
| Earlier | No | Superseded. |

The repository currently contains documentation, repository configuration, licenses, and the sample thumbnail archive. It contains no executable code. When original tooling is published, this table will list the supported tooling versions.

## 2. Reporting a vulnerability

Use GitHub's private vulnerability reporting for this repository:

`https://github.com/GI-STANDARDS/FX3-SOLUTIONS/security/advisories/new`

If private reporting is unavailable, open a public issue titled **"Private security contact requested"** and do not include vulnerability details. The maintainer will arrange a private channel.

A useful report includes:

- The affected file, version, or commit.
- The impact and the attack scenario.
- Reproduction steps or a proof of concept.
- Any suggested fix or mitigation.
- Whether the issue affects the documentation, the repository configuration, or the sample archive.

Do not include game files, extracted assets, account identifiers, private file paths, or third-party proprietary code in a report.

## 3. Scope

| Area | In scope? | Notes |
|------|-----------|-------|
| Repository documentation and links | Yes | Misleading security guidance, unsafe instructions, malicious links. |
| Issue forms, pull request template, and repository configuration | Yes | Injection, unsafe defaults, or data-collection risks. |
| Sample archive integrity | Yes | Malformed archive, path traversal, or tampering. |
| Original tooling, when published | Yes | Code execution, memory safety, unsafe file handling. |
| Far Cry 3, the Dunia 2 engine, or Ubisoft services | No | Report to Ubisoft through its own channels. |
| Decompiled editor source or third-party object packs | No | Report to their authors. |
| Pirated content or DRM bypass | Not accepted | Out of scope. |

## 4. Response

Reports are handled on a best-effort basis. There is no guaranteed response or fix deadline.

The maintainer will acknowledge the report, investigate, and coordinate disclosure with the reporter where practical. Credit is given if the reporter wants it. Please do not test against systems you do not own, and do not publish details before a fix or mitigation is available.

## 5. Safe research

- Work on your own installation and your own backups.
- Do not upload game assets or personal data to a public issue.
- Do not use a vulnerability report to redistribute proprietary material.
- If a test could corrupt a save, map, or installation, back it up first.

## 6. Related documents

- [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)
- [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md)
- [CONTRIBUTING.md](CONTRIBUTING.md)
