# Feature parity matrix

What each MarkNotes app supports today, and where their implementations of the shared format differ. Update this file
in the same pull request that changes behaviour in an app (open the spec PR first, then bump the pin in the app).

Apps: **Android** ([libre](https://github.com/MaxDac/marknotes-android) and
[Pro](https://github.com/MaxDac/MarkNotes), sharing the executor), **Desktop**
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
| Safe sync mutations and crash recovery (sync 0.3 contract) | Yes (0.3 source) | Yes (0.3 source) | Yes (0.3 source) | Source-pinned integrations, not an assumed published release. Unsupported safe mutations fail explicitly; see resolved divergences 32-34 |
| Merge a conflicted copy back, with conflict choices | Yes | Yes | Yes | |
| When sync runs | Launch and Sync now | Launch and Sync now | Page load and Sync now | No app polls in the background |
| Attachment hash check on download | Yes | Yes | Yes | [note-format.md](note-format.md#attachment-sync) |
| Delete local data | No (planned) | Yes | No (planned) | See [Local data and privacy](#local-data-and-privacy). Desktop: [MaxDac/marknotes-electron#35](https://github.com/MaxDac/marknotes-electron/pull/35) |
| Local error diagnostics and user-initiated GitHub reporting | Yes (Libre and Pro) | No | No | Android-only; see [Android diagnostics](#android-diagnostics). App-only, no format change |
| Remote images blocked by default | Attachments only | Yes | Attachments only | Desktop: per-note "Load remote images", `https:` only, for the session. Android and Web never load remote images |
| App lock with idle timeout | No (planned) | No (planned) | No (planned) | Android: biometric or device credential. Desktop and Web: passphrase |
| Local encryption at rest | Yes (device file-based encryption, app-private storage) | No (planned) | No (planned) | Web: only with the app lock on. See [Local data and privacy](#local-data-and-privacy) |
| Links to the other apps in Settings → About | No (planned) | No (planned) | No (planned) | See [Links between apps](#links-between-apps). App-only, no format change |

## Links between apps

App behaviour only; none of it changes the note format, the manifest or the fixtures.

Settings → About in every app, and every app's README, list the other MarkNotes apps so users can find them. Android
Libre and Pro are listed separately, so each app shows exactly three entries and never links to itself. Android
Libre and Pro share the list; each host app tells the shared code which edition it is.

| App | Link | Label |
| --- | --- | --- |
| Android (Libre) | <https://f-droid.org/packages/io.github.maxdac.marknotes.libre/> | MarkNotes for Android (F-Droid) |
| Android Pro | <https://github.com/MaxDac/MarkNotes/releases> | MarkNotes Pro for Android (GitHub Releases) |
| Desktop | <https://github.com/MaxDac/marknotes-electron/releases> | MarkNotes Desktop (GitHub Releases) |
| Web | <https://marknotes.fly.dev> | MarkNotes Web |

- Links open in the external browser (a new tab on Web) only when the user picks one; opening Settings loads
  nothing. The URLs are fixed: no tracking parameters and no note, drive or account data.
- GitHub links point at the releases list, not `/releases/latest`, which is empty while an app has only
  prereleases. Labels say "GitHub Releases" rather than promising a download.
- If the link cannot be opened (no browser, or the system blocks it), the app shows an error.
- The F-Droid listing is not published yet, so that link returns 404 until it is.

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

### Android diagnostics

MarkNotes Libre and Pro share this Android-only behaviour. It does not change the note or sync format, add fixture
requirements or imply Desktop or Web support.

- **Local logging** is on by default for Java/Kotlin crashes and handled errors. Native crashes and ANRs are
  excluded initially. Retention is capped at seven days and 5 MiB total, including export snapshots, with pruning
  on startup, write and export; deletion is not guaranteed while the app is force-stopped.
- **Sanitization** uses allowlisted error codes and failure categories only, excluding all user content, paths,
  identifiers, credentials and raw error messages from stored records, draft excerpts and exports.
- **Clear logs** removes local diagnostics, including export snapshots. The delete-local-data flow must also
  remove them; this requirement does not mark Android's broader delete-local-data feature as implemented.
- **Report an error** primarily opens a GitHub issue draft after explicit consent and review of a sanitized,
  bounded recent-error excerpt. The complete encoded URL is limited to 2,000 ASCII characters; include whole
  records only and indicate truncation when records are omitted. The user submits the issue in the browser.
  Before opening, warn that opening the draft already transmits the URL's logs to GitHub, before submission,
  and can leave them in browser history. Libre targets
  [MaxDac/marknotes-android](https://github.com/MaxDac/marknotes-android/issues/new); Pro targets
  [MaxDac/MarkNotes](https://github.com/MaxDac/MarkNotes/issues/new).
- **Share logs** is a secondary, offline local export of the full retained sanitized logs, subject to the same
  retention and total storage limits. Nothing is sent automatically.

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
29. Resolved in the 0.3 source integrations: all three apps reject duplicate listed paths, preserve a folder omitted
    from a listing that still includes its descendants, and allocate conflicted-copy names using Unicode simple
    case folding (`sync/plan`, `merge/conflict-copy-name`), including the fixes introduced in marknotes-sync 0.2.
30. **Byte order mark** (`front-matter/split`, `front-matter/read-id`, `front-matter/with-id`, `chords/detect`): no
    app recognises a front-matter block after a UTF-8 byte order mark. The related sync-path control-character
    filtering is resolved in all three apps' 0.3 source integrations; front-matter and chord annotations remain.
31. Resolved in the 0.3 source integrations: every app persists `localDeleted` per record and drive, flags records
    for all linked drives after listing the shared local tree, and maps `sync/local-deletions.json`. A restored note
    cannot be deleted through another drive's outdated record
    ([#37](https://github.com/MaxDac/marknotes-spec/issues/37)).
32. Resolved in the 0.3 source integrations: known files are deleted with individual guards, followed by deepest-first
    empty-only folder removal. New files, conflicted copies and planned folders are create-only. Unsupported safe
    removal or creation fails explicitly and retains the affected records instead of adopting or replacing an entry
    (`sync/plan`, `sync/executor-safety`;
    [#38](https://github.com/MaxDac/marknotes-spec/issues/38),
    [#41](https://github.com/MaxDac/marknotes-spec/issues/41)).
33. Resolved in the 0.3 source integrations: record-proven unchanged stale case spellings converge without changing
    content or edit times. When both listed sides already agree, only the current drive's stale history is migrated.
    Rename journals recover interrupted moves and history migration; ambiguous or edited stale sides remain errors
    (`sync/plan`; [#39](https://github.com/MaxDac/marknotes-spec/issues/39)). Provider capability is separate:
    Dropbox does not support a direct case-only drive move, and unsupported safe operations remain explicit step
    errors rather than temporary moves that could look like deletions to peers.
34. Resolved in the 0.3 source integrations: conflicted copies retain the losing edit time, including epoch zero.
    Every executor persists intent and recovery data before mutation, reuses only an exact verified same-conflict
    copy and its replacement id, and repairs interrupted winner/copy uploads, bases and records without erasing
    intervening edits (`sync/reconcile`, `sync/executor-safety`;
    [#40](https://github.com/MaxDac/marknotes-spec/issues/40),
    [#42](https://github.com/MaxDac/marknotes-spec/issues/42)).

Shared algorithm fixtures (`fixtures/sync/*`, `manifest/*`, `merge/*`, except `sync/executor-safety.json`) are run by
marknotes-sync for every app. App runners may map those algorithms to the library, but must still map every file.
`sync/executor-safety.json` instead exercises each actual app executor with injected races and durable restart
boundaries, reporting observed store state rather than returning the expected object. Apps also keep their own
SDK-facing contract tests for the [drive adapters](note-format.md#drive-adapter-contract).
