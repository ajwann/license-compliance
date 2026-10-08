---
name: license-compliance
description: Inventory the licenses of a project's dependencies (direct and transitive) with real per-ecosystem tools (pip-licenses, license-checker-rseidelsohn, cargo-deny, go-licenses, and others), judge each against how the product actually ships (internal tool, SaaS, distributed binary or container, library, or open-source project), resolve SPDX OR/AND expressions, flag copyleft, AGPL, source-available, non-commercial, and missing licenses, and generate a THIRD_PARTY_NOTICES/attribution file. Use whenever the user asks about licenses, "can we use this package commercially", GPL/AGPL/copyleft concerns, open-source compliance, an OSS or legal review, SBOM-adjacent questions, attribution or NOTICE files, before shipping or open-sourcing a product, or when adding a dependency with an unusual license. Gives engineering triage, not legal advice; it hands real ambiguities to a human.
---

# License compliance

The same license can be fine in one product and a blocker in another.
AGPL in an internal CLI is harmless. AGPL in a SaaS backend means publishing
your source. A scan that just prints "GPL found!" without knowing how the
product ships is noise. This skill pairs the inventory with the distribution
model, so every flag means something.

This is engineering triage, not legal advice. State that once in the report,
and route real ambiguities to a person with the facts attached, rather than
making a judgment call yourself.

## 1. Establish the distribution model

Ask the user or infer it from the repo (Dockerfile pushed to a customer
registry? Published to npm or PyPI? Only a web service?), and say which you
assumed:

- internal only
- SaaS / hosted
- distributed binary, app, firmware, or container image
- library or SDK others embed
- open source under license X

Also note which dependencies end up in the shipped artifact and which are
dev/build only.

## 2. Inventory

Install the scanner into a throwaway environment or run it with
`npx`/`uvx` so it doesn't pollute the project. Scan the **installed or
resolved** tree, because lockfiles bring in transitive packages the manifest
doesn't list.

| Ecosystem | Command |
|---|---|
| Python | `pip-licenses --from=mixed --format=markdown --with-urls` (run inside the project's venv; `--python <venv>/bin/python` targets another env). `--allow-only="MIT;BSD License;…"` / `--fail-on=` for CI |
| Node | `npx license-checker-rseidelsohn --production --summary`, then `--json` for detail. `--onlyAllow "MIT;ISC;…"` / `--failOn` for CI |
| Rust | `cargo deny init`, then set `[licenses] allow = [...]` in `deny.toml`, then `cargo deny check licenses` |
| Go | `go-licenses report ./... > licenses.csv`, and `go-licenses check ./...` (see `go-licenses help check` for allow/deny flags) |
| Java | Maven `license-maven-plugin` (`mvn license:aggregate-third-party-report`); Gradle `com.github.jk1.dependency-license-report` |
| Ruby | `license_finder` |
| .NET | `dotnet-project-licenses` (nuget-license) |
| Containers / mixed | `syft <image> -o spdx-json` for an SBOM with licenses, then review it with the table below |

Check flags with `--help` on the installed version. These tools change.

## 3. Classify and decide

Normalize every license to SPDX. Then, using `references/compatibility.md`
(families × distribution model, plus SPDX expression rules):

- **ok**: permissive, or copyleft that doesn't apply to this distribution model.
- **ok with obligation**: e.g. Apache-2.0 NOTICE files, LGPL relinking, MPL
  file-level source sharing if modified.
- **blocking**: e.g. GPL in a distributed proprietary binary, AGPL in SaaS.
  Propose an alternative package or an isolation approach, but leave the
  legal judgment to a human.
- **review**: source-available, non-commercial, custom, missing, or
  contradictory licenses (the metadata says MIT but the LICENSE file says GPL). Read the actual
  LICENSE file in the package or repo before concluding. Metadata is wrong
  surprisingly often.

For `A OR B`, choose the option that fits and record the choice. For
`A AND B`, both apply.

## 4. Generate attribution (if shipping)

Distributing permissive dependencies still requires including their
copyright notices. Generate `THIRD_PARTY_NOTICES.md` (or the format the
project uses) with the package, version, license, copyright line, and full
license text, deduplicated by license text. Sources:
`pip-licenses --with-license-file --with-notice-file --format=json`,
`license-checker-rseidelsohn --files <dir>`, `go-licenses save`, and
`cargo about` for Rust. Include Apache-2.0 `NOTICE` file contents verbatim.

## 5. Report

```
Distribution model: SaaS (assumed from: Dockerfile deploys to our ECS; nothing published)
Scanned: 214 packages (Python 3.12 venv + npm production tree)  ·  Not legal advice.

BLOCKING
  - some-pdf-lib 3.1 (AGPL-3.0-only), imported by app/export.py
      → alternatives: pypdf (BSD-3-Clause); or isolate it behind a separate service and get a legal review
REVIEW
  - fancy-charts 2.0: metadata "UNKNOWN"; repo LICENSE is BUSL-1.1 (no competing hosted use)
OK WITH OBLIGATIONS
  - 12 × Apache-2.0: NOTICE files must ship with distributed artifacts (n/a for SaaS)
OK
  - 196 packages: MIT 120, BSD-3 41, ISC 22, Apache-2.0 12, MPL-2.0 1 (unmodified)
CI guard: added `--onlyAllow` list matching the OK set
```
