# Copilot instructions

Follow [AGENTS.md](../AGENTS.md). In short: this repository is the shared MarkNotes spec. Keep `note-format.md`,
`fixtures/` and `features.md` consistent in the same change, never loosen an `expected` value to match one app (add a
`knownDivergence` entry instead), and keep fixture files valid against `schemas/fixture-file.schema.json`.

For cross-repository ownership, package consumption or release work, use [the shared handbook](../docs/agents.md)
and the project skills under `.claude/skills/`. Keep common procedures here and local command maps in consumers;
do not assume a published upstream release is already consumed or deployed.
