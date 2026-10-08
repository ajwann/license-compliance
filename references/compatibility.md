# License obligations by distribution model

This is a working engineer's map, not legal advice. Anything marked
**review** goes to a human (legal, the OSPO, or the project owner) along with
the facts: package, license, how it's linked, and how the product ships.

## Families

| Family | Common SPDX IDs | Core obligation |
|---|---|---|
| Permissive | MIT, BSD-2-Clause, BSD-3-Clause, ISC, Apache-2.0, Zlib, 0BSD, Unlicense, CC0-1.0 | Keep the copyright and license text (NOTICE/attribution). Apache-2.0 also requires keeping its `NOTICE` file and stating changes, and includes a patent grant |
| Weak copyleft (file or library) | MPL-2.0, LGPL-2.1, LGPL-3.0, EPL-2.0, CDDL-1.0 | Modifications *to those files or that library* must be shared under the same license. Using it unmodified is usually fine. LGPL also requires that users can relink or replace the library: easy with dynamic linking, harder with static |
| Strong copyleft | GPL-2.0, GPL-3.0 | If you **distribute** a combined work, the whole work must be released under GPL, with source |
| Network copyleft | AGPL-3.0 | Like GPL, but users interacting over a **network** count as distribution. Applies to SaaS |
| Source-available / non-OSI | BUSL-1.1, SSPL-1.0, Elastic-2.0, Commons Clause, PolyForm-*, RSAL | Use restrictions, often "no competing hosted service". **Review** |
| Non-commercial / custom | CC-BY-NC-*, "free for personal use", JSON license ("shall be used for Good, not Evil") | Often blocks commercial use. **Review** |
| Content licenses on code | CC-BY-SA-4.0, CC-BY-4.0 | Fine for docs and assets; unusual on code. **Review** if found on code |
| Missing / UNKNOWN / NOASSERTION | — | No license means all rights reserved by default. **Review**; check the repo for a LICENSE file the metadata missed |

## Which ones matter for how the product ships

| Distribution model | Permissive | Weak copyleft | GPL | AGPL | Source-available |
|---|---|---|---|---|---|
| Internal tool, never leaves the company | ok | ok | ok (no distribution) | ok, unless network users are outside the company | review terms |
| SaaS / hosted service (code runs on your servers) | ok + attribution | ok | ok (no distribution of binaries) | **blocking unless you'll publish source** | **review**: many forbid offering it as a service |
| Distributed binary, desktop/mobile app, firmware, container image given to customers | ok + attribution | ok if unmodified and the relink requirement is met | **blocking** for proprietary products | **blocking** | review |
| Published library / SDK others embed | ok | ok, but it passes the obligation to your users; mention it | makes your library effectively GPL | same, stronger | review |
| Open-source project under license X | Must be compatible with X: e.g. Apache-2.0 code can go into GPL-3.0, not GPL-2.0-only | | | | |

Dev-only dependencies (test runners, linters, build tools) that don't end up
in the shipped artifact usually carry no distribution obligations. Check that
they really don't end up there: a bundler can pull dev dependencies in.

## SPDX expressions

- `MIT OR Apache-2.0`: **you choose**. Pick the one that fits and record the choice.
- `MIT AND BSD-3-Clause`: **both** apply (parts under each).
- `GPL-2.0-only` vs. `GPL-2.0-or-later`: matters for compatibility with
  GPL-3.0 and Apache-2.0.
- `WITH Classpath-exception-2.0`, `WITH LLVM-exception`: exceptions that
  usually permit linking without copyleft spreading. Read them.
- Tools often report Python trove classifiers ("BSD License",
  "Mozilla Public License 2.0 (MPL 2.0)") instead of SPDX IDs. Normalize them,
  and for vague ones like "BSD License", read the actual LICENSE file.
