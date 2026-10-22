# Agent instructions

This repository is the single source of truth for behaviour shared by the three MarkNotes apps. It contains no app
code; read [README.md](README.md) first.

## Sibling repositories

| App | Repository | Fixture key | Language |
| --- | --- | --- | --- |
| Android | MaxDac/MarkNotes | `android` | Kotlin |
| Desktop (Electron) | MaxDac/marknotes-electron | `desktop` | TypeScript |
| Web | MaxDac/marknotes-web | `web` | Elixir/Phoenix + TypeScript |

Each app pins this repository as a git submodule at `spec/` and runs the fixtures in a CI job named
`Parity (marknotes-spec fixtures)`.

## Rules

- Change `note-format.md`, the matching fixtures and `features.md` in the same pull request.
- Fixture `expected` values describe the agreed behaviour, not whatever one app does. When an app differs, add a
  `knownDivergence` entry for that app and a line in `features.md` under "Known divergences".
- Keep case names unique within a file; runners report failures by file and case name.
- New fixture files must follow `schemas/fixture-file.schema.json` and be listed in the README table. Tell the app
  maintainers, because runners fail on unmapped files.
- Before changing an expected value, check the current implementation in all three apps (for example with the
  GitHub CLI: `gh api repos/MaxDac/<repo>/contents/<path> -H "Accept: application/vnd.github.raw"`).

## Checks

CI runs `Validate fixtures` (JSON Schema), `actionlint` and `Markdown lint`. Locally:

```sh
pipx run check-jsonschema --schemafile schemas/fixture-file.schema.json fixtures/*/*.json
npx markdownlint-cli2 "**/*.md"
```
