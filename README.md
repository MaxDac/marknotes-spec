# marknotes-spec

The shared specification of the MarkNotes apps:

- [Android](https://github.com/MaxDac/marknotes-android) and its [Pro edition](https://github.com/MaxDac/MarkNotes)
- [Desktop (Electron)](https://github.com/MaxDac/marknotes-electron)
- [Web](https://github.com/MaxDac/marknotes-web)

They share one sync implementation, [marknotes-sync](https://github.com/MaxDac/marknotes-sync) (Rust, with Kotlin
and WebAssembly bindings), which runs the sync fixtures here.
The Android editions share their executor in marknotes-android; the Pro repository pins it at `libre/` and adds
OneDrive and Proton Drive adapters. Both use the `android` fixture key.

All editions read and write the same notes-folder format and interoperate through their supported drives,
so they must agree on the shared behavior. Provider availability differs by edition; see [features.md](features.md).

All MarkNotes repositories carry the `marknotes` topic (list them with `gh repo list MaxDac --topic marknotes`),
and their issues and pull requests are tracked together on the
[MarkNotes project](https://github.com/users/MaxDac/projects/16).

| Path | Content |
| --- | --- |
| [note-format.md](note-format.md) | The canonical note, link, attachment, sync, manifest, merge and chord-sequence format |
| [features.md](features.md) | Feature parity matrix and known divergences between the apps |
| [docs/help.md](docs/help.md) | Shared first-access guide content, current Markdown coverage and onboarding acceptance criteria |
| [fixtures/](fixtures) | Golden test cases every app runs against its own implementation |
| [schemas/](schemas) | JSON Schema the fixture files are validated against |
| [docs/architecture.md](docs/architecture.md) | Repository/module graph, runtime boundaries and change ownership |
| [docs/dependencies.md](docs/dependencies.md) | Package channels, source/binary pins, build tools and verification boundaries |
| [docs/releasing.md](docs/releasing.md) | Release ordering, consumer acceptance and manual downstream handoffs |
| [docs/agents.md](docs/agents.md) | Shared Copilot/Claude skills, local entrypoints and discovery/authorization rules |

These operational guides complement the format specification; they do not redefine fixture behavior.
Start with the architecture map when deciding which repository owns a change.

## Fixtures

Each file under `fixtures/<area>/<function>.json` has this shape:

```json
{
  "description": "What the function does and how to read input/expected.",
  "cases": [
    {
      "name": "unique within the file",
      "input": {},
      "expected": {},
      "knownDivergence": { "android": "why Android differs" }
    }
  ]
}
```

| File | Function under test |
| --- | --- |
| `note-ids/validation` | Is a string a valid note id |
| `front-matter/read-id`, `with-id`, `split`, `with-body` | Read the id, set the id, split front-matter from the body, re-attach it on save |
| `note-links/link-markdown`, `parse-href`, `mention`, `filter` | Insert a link, read a link target, detect `@`, filter the picker |
| `attachments/extension-for-type`, `is-attachment-path`, `image-markdown`, `scaled-size`, `refs` | Image attachments |
| `manifest/parse`, `serialize`, `build` | `.marknotes/manifest.json` |
| `duplicate-ids/resolve` | Which notes get a new id when ids collide |
| `merge/merge3`, `conflict-copy-name` | Three-way merge and conflict copies |
| `sync/plan`, `reconcile`, `multi-drive`, `local-deletions` | Sync plan for one drive, notes changed on both sides, order of drives, records to flag as deleted locally |
| `sync/executor-safety` | Safe folder deletion and deferred-removal notice lifecycles, create-only races, and conflicted-copy restart recovery |
| `chords/detect`, `parse`, `serialize`, `validate`, `display-names` | Chord sequences: detect `type: chords`, parse and write the body, validate before saving, number repeated chords |

## How apps consume this repository

1. Direct consumers add it as a git submodule at `spec/`, pinned to a commit on `main`:
   `git submodule add https://github.com/MaxDac/marknotes-spec spec`.
   Pro instead consumes the pin already owned by Libre at `libre/spec/`; change it upstream in Libre,
   then adopt the reviewed Libre commit. Do not create an independent Pro spec pin.
2. Run every fixture file with the app's own test runner, in a CI job named `Parity (marknotes-spec fixtures)`.
   Checkout must initialize the fixture pin: `submodules: true` for direct `spec/` consumers and
   `submodules: recursive` for Pro's nested `libre/spec/`. Ordinary Android unit tests can skip absent fixtures;
   use the required parity tasks documented in the consumer, not unit-test success as parity evidence.
3. For each case, compare the app's output with `expected`. When `knownDivergence` has the app's key (`android`,
   `desktop` or `web`), the runner must assert that the output is **different** from `expected` and report it as a
   known divergence, so that a fix is noticed and the entry removed.
4. Fail on fixture files the runner does not map, so new fixtures are not silently skipped.
5. Let Dependabot (`gitsubmodule` ecosystem) open pull requests that bump the pin.

The apps' `docs/note-format.md` should link here instead of keeping a copy.

`sync/executor-safety` runs against marknotes-sync's reference executor, because planner inputs cannot express a
provider race or a process restart. App runners must map it, not silently ignore it. Desktop and Web run its cases
against their own executors, as does the current Android runner. Pro gets that runner through its pinned Libre
source, so check the consumed pin rather than assuming it matches Libre main. Apps also keep their own adapter
and durable-store tests for the same guarantees.

The deferred-folder cases for [#50](https://github.com/MaxDac/marknotes-spec/issues/50) intentionally change the
unsupported-capability result and add executor scenarios to this existing file. The marknotes-sync Rust reference
executor and its Kotlin/npm observation mappings, Desktop and Web executor runners, and Android's explicit mapping
must be updated in coordination before consumers bump this pin. Unknown scenarios are not known-divergence
observations: runners must execute a setup and compare its actual output, not skip it, return its golden result or
catch an unmapped-scenario exception as an expected divergence. Current-main app divergences remain annotated until
the corresponding implementation is verified; the reference executor must satisfy every expected result.

## Changing the spec

1. Open a pull request here that updates `note-format.md`, the fixtures and `features.md` together.
2. After it merges, bump direct consumers' spec pins and fix or mark the cases that now fail.
   Coordinate shared sync changes in marknotes-sync, and have Pro adopt the reviewed Libre pin rather than
   editing its nested spec independently. See [release ordering](docs/releasing.md).
3. Never loosen an `expected` value to match one app; add `knownDivergence` instead and agree on the fix.
