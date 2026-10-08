# license-compliance

An [Agent Skill](https://agentskills.io): **license compliance**.

Inventory the licenses of a project's dependencies (direct and transitive) with real per-ecosystem tools (pip-licenses, license-checker-rseidelsohn, cargo-deny, go-licenses, and others), judge each against how the product actually ships (internal tool, SaaS, distributed binary or container, library, or open-source project), resolve SPDX OR/AND expressions, flag copyleft, AGPL, source-available, non-commercial, and missing licenses, and generate a THIRD_PARTY_NOTICES/attribution file.

## Install

```bash
npx skills add ajwann/license-compliance
```

Or copy this directory to `~/.claude/skills/license-compliance/` (Claude Code) or your agent's skills folder.

## Contents

- `SKILL.md`: the instructions the agent loads (the frontmatter `description` says when it triggers)
- `references/compatibility.md`

## License

MIT
