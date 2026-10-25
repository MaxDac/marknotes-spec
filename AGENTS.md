# Agent instructions

This repository is the single source of truth for behaviour shared by the MarkNotes apps. It contains no app
code; read [README.md](README.md) first.

For ecosystem work, read [architecture](docs/architecture.md), [dependency consumption](docs/dependencies.md),
[release coordination](docs/releasing.md) and [agent workflows](docs/agents.md). Common procedures live in
`.claude/skills/marknotes-dependencies/SKILL.md` and `.claude/skills/marknotes-release/SKILL.md`; local app
entrypoints provide path/command maps rather than copied procedures. Do not assume nested submodule skills
are automatically discovered.

## Sibling repositories

| App | Repository | Fixture key | Language |
| --- | --- | --- | --- |
| Android Libre | MaxDac/marknotes-android | `android` | Kotlin |
| Android Pro (pins Libre at `libre/`) | MaxDac/MarkNotes | `android` | Kotlin |
| Desktop (Electron) | MaxDac/marknotes-electron | `desktop` | TypeScript |
| Web | MaxDac/marknotes-web | `web` | TypeScript + Vite (static PWA served by Caddy) |
| Shared sync library | MaxDac/marknotes-sync | (none: implements every case) | Rust, Kotlin and WASM bindings |

Each app pins this repository as a git submodule at `spec/` (Pro through Libre's pin at `libre/spec/`) and runs the
fixtures in a CI job named `Parity (marknotes-spec fixtures)`. Pin commits that are on `main`. marknotes-sync pins it
too and must pass every sync, manifest and merge case; change the sync rules there in the same round as here.

All of them carry the `marknotes` topic (`gh repo list MaxDac --topic marknotes`) and are linked to the
[MarkNotes project](https://github.com/users/MaxDac/projects/16). Add cross-repository issues and pull requests to it
(`gh project item-add 16 --owner MaxDac --url <url>`) and set its `Repo area` field.

## Rules

- Change `note-format.md`, the matching fixtures and `features.md` in the same pull request.
- Fixture `expected` values describe the agreed behaviour, not whatever one app does. When an app differs, add a
  `knownDivergence` entry for that app and a line in `features.md` under "Known divergences".
- Keep case names unique within a file; runners report failures by file and case name.
- New fixture files must follow `schemas/fixture-file.schema.json` and be listed in the README table. Tell the app
  maintainers, because runners fail on unmapped files.
- Before changing an expected value, check the shared sync implementation and all affected app editions, including
  Pro's actual pinned Libre tree rather than assuming it matches Libre main (for example with the GitHub CLI:
  `gh api repos/MaxDac/<repo>/contents/<path> -H "Accept: application/vnd.github.raw"`).
- Operational documentation and validation changes do not require unrelated fixture/format edits. If a shared
  behavior changes, update the format, matching fixtures and feature matrix together as above.
- Distinguish published packages, consumed source/binary pins and deployed app revisions. Pro consumes shared
  Libre source/catalog but owns its Gradle root; Web/Desktop consume sync Git source, Android uses release assets
  or an explicit source build. Record the actual pinned trees, not only upstream main.
- No skill or documentation plan implicitly authorizes dependency upgrades, PR creation, merges, tags,
  publication, deployment or destructive rollback. Web main merges can deploy production.

## Checks

CI runs `Validate fixtures` (JSON Schema), `actionlint` and `Markdown lint`. Locally:

```sh
pipx run check-jsonschema --schemafile schemas/fixture-file.schema.json fixtures/*/*.json
npx markdownlint-cli2 "**/*.md"
```
