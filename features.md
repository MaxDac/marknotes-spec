# Feature parity matrix

What each MarkNotes app supports today, and where their implementations of the shared format differ. Update this file
in the same pull request that changes behaviour in an app (open the spec PR first, then bump the pin in the app).

Apps: **Android** ([MaxDac/MarkNotes](https://github.com/MaxDac/MarkNotes)), **Desktop**
([MaxDac/marknotes-electron](https://github.com/MaxDac/marknotes-electron)) and **Web**
([MaxDac/marknotes-web](https://github.com/MaxDac/marknotes-web)). Fixture divergence keys are `android`, `desktop` and
`web`.

## Features

| Feature | Android | Desktop | Web | Notes |
| --- | --- | --- | --- | --- |
| Markdown notes in nested folders | Yes | Yes | Yes | [note-format.md](note-format.md#notes-and-folders) |
| Chord sequence editor | Yes | Yes | Yes | [note-format.md](note-format.md#chord-sequences); `fixtures/chords/*` |
| Chord sequences opt-in setting (off by default) | Yes | Yes | Yes | [note-format.md](note-format.md#chord-sequences-setting). App-only, no format change |
| Stable note ids in front-matter | Yes | Yes | Yes | |
| Note links and `@` picker | Yes | Yes | Yes | Picker ordering differs, see below |
| Images from file picker / toolbar | Yes | Yes | Yes | |
| Image paste and drag-and-drop | No | Yes | Yes | Android: file picker and camera only |
| Unused attachment clean-up | Yes | Yes | Yes | |
| Import shared text and documents as Markdown | Yes | No | No | Android share sheet: text, HTML, `.txt`/`.md`/`.html`, `.docx`, PDF text, images. App-only, no format change |
| Dropbox sync | Yes | Yes | Yes | |
| OneDrive sync | Yes | Yes | Yes | |
| Proton Drive sync | Yes | Yes | No | Web: deferred to marknotes-web#1 |
| Shared sync algorithm ([marknotes-sync](https://github.com/MaxDac/marknotes-sync)) | Yes | Yes | Yes | [note-format.md](note-format.md#sync); `fixtures/sync/*` |
| Sync manifest (`.marknotes/manifest.json`) | Yes | Yes | Yes | |
| Three-way merge, newest wins on conflict | Yes | Yes | Yes | The other version is saved as a conflicted copy |
| Merge a conflicted copy back, with conflict choices | Yes | Yes | Yes | |
| When sync runs | Launch and Sync now | Launch and Sync now | Page load and Sync now | No app polls in the background |
| Attachment hash check on download | Yes | Yes | Yes | [note-format.md](note-format.md#attachment-sync) |
| Delete local data | No (planned) | Yes | No (planned) | See [Local data and privacy](#local-data-and-privacy). Desktop: [MaxDac/marknotes-electron#35](https://github.com/MaxDac/marknotes-electron/pull/35) |
| Remote images blocked by default | Attachments only | Yes | Attachments only | Desktop: per-note "Load remote images", `https:` only, for the session. Android and Web never load remote images |
| App lock with idle timeout | No (planned) | No (planned) | No (planned) | Android: biometric or device credential. Desktop and Web: passphrase |
| Local encryption at rest | Yes (device file-based encryption, app-private storage) | No (planned) | No (planned) | Web: only with the app lock on. See [Local data and privacy](#local-data-and-privacy) |

## Local data and privacy

App behaviour only; none of it changes the note format, the manifest or the fixtures.

- **Delete local data** removes the notes, images and sync state stored on the device. It is offered when the last
  drive is disconnected ("Also delete the notes stored on this device") and as Settings → Privacy → "Delete all local
  data", which disconnects every drive first. The local tree is shared by all drives, so it is deleted only when no
  drive is connected, and the deletion never propagates to any drive.
- **Remote images**: only note attachments render automatically. `https:` images in note bodies stay blocked until the
  user loads them for that note; `http:`, protocol-relative (`//host`) and `file:` images are never loaded.
- **App lock**: optional, with an idle timeout, and also locks on system suspend or screen lock where the platform
  reports it.
- **Encryption at rest**: Desktop encrypts the local note database with a data key protected by the OS keychain, or by
  the app-lock passphrase when set. Web encrypts every local record, paths included, only when the app lock is
  enabled, with a key derived from the passphrase (PBKDF2-SHA256, 600,000 iterations). Android adds no app-level
  cipher: with minSdk 29, file-based encryption is mandatory, so notes in app-private storage are already encrypted
  under the device credential and sandboxed from other apps.

## Known divergences

Each item is either a bug to fix in the listed app or a spec change to agree on. Fixture cases that cover them carry a
`knownDivergence` entry; remove it when the app is fixed.

1. **Reading the id** (`front-matter/read-id`): Desktop accepts only a valid id. Android returns any non-empty value
   (one pair of quotes stripped); Web accepts any token without whitespace or quotes.
2. **Replacing an invalid id line** (`front-matter/with-id`): Web keeps the invalid `id:` line and adds a second one.
3. **Opening fence** (`front-matter/read-id`): Android requires exactly `---`; Desktop and Web allow trailing spaces or
   tabs. No app handles a UTF-8 BOM.
4. **Line endings when adding an id**: Android uses CRLF only if the block starts with `---\r\n`; Desktop and Web use
   CRLF if the block contains `\r\n` anywhere.
5. **Link hrefs** (`note-links/parse-href`): Web matches `marknotes:` case-sensitively and does not trim, lowercase or
   validate the id; Android does not validate the id.
6. **Link text** (`note-links/link-markdown`): only Desktop replaces line breaks with a space.
7. **Picker ordering** (not in fixtures): Android lists id-prefix matches first, then other matches, each sorted by
   lowercased path; Desktop and Web keep the index order (path, code-unit order). Desktop ignores queries longer than
   80 characters.
8. **Image alt text** (`attachments/image-markdown`): Desktop replaces `[`, `]`, `\` and line breaks with spaces;
   Android and Web escape them. For a name such as `.png`, Web produces empty alt text.
9. **Attachment names** (`attachments/is-attachment-path`, `attachments/refs`): Android accepts any name with a
   supported extension and uses the broad reference pattern `attachments/[A-Za-z0-9._-]+`.
10. **MIME parameters** (`attachments/extension-for-type`): Android strips parameters such as `; charset=...`.
11. Resolved: all three apps reject attachment downloads whose bytes do not match their name
    ([#12](https://github.com/MaxDac/marknotes-spec/issues/12)).
12. **Duplicate ids**: Android compares ids case-insensitively; Desktop and Web compare them exactly.
13. Resolved: manifest parsing on Web now uses marknotes-sync.
14. **MIME case** (`attachments/extension-for-type`): Web matches MIME types case-sensitively.
15. Resolved: manifest parsing on Android now uses marknotes-sync.
16. **Mention detection on Android** (`note-links/mention`): Android opens the picker for `@` followed by a space and
    closes it only when the query has no results.
17. **Missing function on Web**: no note id validator (`note-ids/validation`); the Web parity runner skips that file.
    The three-way merge now comes from marknotes-sync.
18. Resolved: all three apps implement chord sequences and run `fixtures/chords/*`.
19. Resolved: all three apps insert the missing line break when re-attaching front-matter (`front-matter/with-body`).
20. Resolved: all three apps re-hash a local file before writing or deleting it during a sync, and uploads record the
    hash of the uploaded bytes ([#9](https://github.com/MaxDac/marknotes-spec/issues/9)).
21. Resolved: all three apps delete drive files conditionally on their revision
    ([#10](https://github.com/MaxDac/marknotes-spec/issues/10)).
22. Resolved: all three apps use the local edit time as the modification time, on uploads, listings and downloads
    ([#11](https://github.com/MaxDac/marknotes-spec/issues/11)).
23. Resolved: all three apps count local folder creation as a local change
    ([#15](https://github.com/MaxDac/marknotes-spec/issues/15)).
24. Resolved: a failed OneDrive full listing no longer leaves a delta checkpoint, and an undecryptable Proton Drive
    name fails the listing ([#16](https://github.com/MaxDac/marknotes-spec/issues/16),
    [#17](https://github.com/MaxDac/marknotes-spec/issues/17)).
25. Resolved: all three apps save the conflicted copy before overwriting the losing version
    ([#18](https://github.com/MaxDac/marknotes-spec/issues/18)).
26. Resolved: all three apps persist pending conflicts across restarts
    ([#22](https://github.com/MaxDac/marknotes-spec/issues/22)).
27. Resolved: Desktop uploads attachments to Proton Drive with their image media type
    ([MaxDac/marknotes-electron#31](https://github.com/MaxDac/marknotes-electron/issues/31)).
28. Resolved: all three apps run `sync/plan`, `sync/reconcile` and `manifest/parse` against marknotes-sync 0.1.1
    ([#8](https://github.com/MaxDac/marknotes-spec/issues/8), [#14](https://github.com/MaxDac/marknotes-spec/issues/14),
    [#19](https://github.com/MaxDac/marknotes-spec/issues/19), [#20](https://github.com/MaxDac/marknotes-spec/issues/20),
    [MaxDac/marknotes-sync#8](https://github.com/MaxDac/marknotes-sync/issues/8)).
29. **marknotes-sync 0.2.0** (`sync/plan`, `merge/conflict-copy-name`): the apps run marknotes-sync 0.1, which lets
    the last of several entries for the same path win, deletes a folder that a listing omits although it lists
    entries inside it, and picks conflicted-copy names case-insensitively by lowercasing instead of by simple case
    folding. Resolved when each app picks up marknotes-sync 0.2.0.
30. **Byte order mark and control characters** (`front-matter/split`, `front-matter/read-id`, `front-matter/with-id`,
    `chords/detect`, `sync/plan`): no app recognises a front-matter block after a UTF-8 byte order mark, and the apps
    on marknotes-sync 0.1 sync names with control characters. Resolved when each app handles the byte order mark and
    picks up marknotes-sync 0.2.0.
31. **Records flagged as deleted locally** (`sync/plan`, `sync/local-deletions`): marknotes-sync 0.2 has no
    `localDeleted` flag and no `local_deletions`, and no app flags records, so with several drives a note moved away
    and back can be deleted everywhere through another drive's out-of-date record
    ([#37](https://github.com/MaxDac/marknotes-spec/issues/37)). Resolved when marknotes-sync implements the flag
    (0.3.0) and each app stores it per record and drive, flags the records of every drive after listing the local tree,
    and maps the new `sync/local-deletions.json` fixture file.

Sync fixtures (`fixtures/sync/*`, `manifest/*`, `merge/*`) are run by marknotes-sync, which implements them for every
app. App runners may map them to the library instead of running them again, but must still map the files. Apps keep
their own contract tests for the [drive adapters](note-format.md#drive-adapter-contract), which no fixture reaches.
