# marknotes-spec

The shared specification of the MarkNotes apps:

- [Android](https://github.com/MaxDac/MarkNotes)
- [Desktop (Electron)](https://github.com/MaxDac/marknotes-electron)
- [Web](https://github.com/MaxDac/marknotes-web)

They share one sync implementation, [marknotes-sync](https://github.com/MaxDac/marknotes-sync) (Rust, with Kotlin
and WebAssembly bindings), which runs the sync fixtures here.

All three read and write the same notes folder and sync it through the same drives, so they must agree on the format.

All MarkNotes repositories carry the `marknotes` topic (list them with `gh repo list MaxDac --topic marknotes`),
and their issues and pull requests are tracked together on the
[MarkNotes project](https://github.com/users/MaxDac/projects/16).

| Path | Content |
| --- | --- |
| [note-format.md](note-format.md) | The canonical note, link, attachment, sync, manifest, merge and chord-sequence format |
| [features.md](features.md) | Feature parity matrix and known divergences between the apps |
| [fixtures/](fixtures) | Golden test cases every app runs against its own implementation |
| [schemas/](schemas) | JSON Schema the fixture files are validated against |

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
| `chords/detect`, `parse`, `serialize`, `validate`, `display-names` | Chord sequences: detect `type: chords`, parse and write the body, validate before saving, number repeated chords |

## How apps consume this repository

1. Add it as a git submodule at `spec/`, pinned to a commit on `main`:
   `git submodule add https://github.com/MaxDac/marknotes-spec spec`.
2. Run every fixture file with the app's own test runner, in a CI job named `Parity (marknotes-spec fixtures)`.
   Checkout must use `submodules: true`.
3. For each case, compare the app's output with `expected`. When `knownDivergence` has the app's key (`android`,
   `desktop` or `web`), the runner must assert that the output is **different** from `expected` and report it as a
   known divergence, so that a fix is noticed and the entry removed.
4. Fail on fixture files the runner does not map, so new fixtures are not silently skipped.
5. Let Dependabot (`gitsubmodule` ecosystem) open pull requests that bump the pin.

The apps' `docs/note-format.md` should link here instead of keeping a copy.

## Changing the spec

1. Open a pull request here that updates `note-format.md`, the fixtures and `features.md` together.
2. After it merges, bump the submodule in each app and fix or mark the cases that now fail.
3. Never loosen an `expected` value to match one app; add `knownDivergence` instead and agree on the fix.
