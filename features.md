# Feature parity matrix

What each MarkNotes app supports today, and where their implementations of the shared format differ. Update this file
in the same pull request that changes behaviour in an app (open the spec PR first, then bump the pin in the app).

Apps: **Libre** ([MaxDac/marknotes-android](https://github.com/MaxDac/marknotes-android)), **Pro**
([MaxDac/MarkNotes](https://github.com/MaxDac/MarkNotes), which pins Libre at `libre/` and adds OneDrive and Proton
Drive), **Desktop** ([MaxDac/marknotes-electron](https://github.com/MaxDac/marknotes-electron)) and **Web**
([MaxDac/marknotes-web](https://github.com/MaxDac/marknotes-web), TypeScript + Vite, served by Caddy). Libre and Pro
share their editor, settings and sync executor, so they share the `android` fixture key; the other keys are `desktop`
and `web`.

"Yes" means shipped on the default branch, read in the source; a linked issue tracks every user-facing gap. Last full
audit: 2026-10-24, at marknotes-android `24a84ee`, MarkNotes `ebdf80d` (Libre pin `76e0221`), marknotes-electron
`a416c8e` and marknotes-web `53c5be4`.

Verified default-branch updates after that audit: Libre's reserved-root move guard in
[MaxDac/marknotes-android#33](https://github.com/MaxDac/marknotes-android/pull/33) (`130c441`) and Desktop's bass
fret-12 marker in [MaxDac/marknotes-electron#49](https://github.com/MaxDac/marknotes-electron/pull/49) (`e601472`).
Pro's checked Libre pin `d0203b9` does not yet include the reserved-root move fix.
Rows that cite a pull request were updated from that pull request, as merged during the 2026-10-24 issue round, not
from a new full audit.
Cross-app links are also merged in
[MaxDac/marknotes-android#23](https://github.com/MaxDac/marknotes-android/pull/23),
[MaxDac/MarkNotes#37](https://github.com/MaxDac/MarkNotes/pull/37),
[MaxDac/marknotes-electron#42](https://github.com/MaxDac/marknotes-electron/pull/42) and
[MaxDac/marknotes-web#45](https://github.com/MaxDac/marknotes-web/pull/45).

## Features

Search, tags, home-screen widgets and background sync while the app is closed are out of scope for every app.

### Notes and editor

| Feature | Libre | Pro | Desktop | Web | Notes |
| --- | --- | --- | --- | --- | --- |
| Markdown notes in nested folders | Yes | Yes | Yes | Yes | [note-format.md](note-format.md#notes-and-folders) |
| Note title is the file name without `.md` | Yes | Yes | Yes | Yes ([MaxDac/marknotes-web#55](https://github.com/MaxDac/marknotes-web/pull/55)) | See known divergence 36 |
| `attachments` reserved as a root name | Yes | Partial: awaiting the Libre move-guard pin | Yes | Yes | On create, rename and move. See known divergence 37 |
| Folder tree with current location | Drawer, folder name | Drawer, folder name | Sidebar, folder name | Sidebar, breadcrumb | |
| Editor views | Edit or Preview | Edit or Preview | Edit, Split or Preview | Edit, Split or Preview ([MaxDac/marknotes-web#57](https://github.com/MaxDac/marknotes-web/pull/57)) | Desktop and Web open each note in Split |
| Formatting toolbar | Yes | Yes | Yes | Yes ([MaxDac/marknotes-web#59](https://github.com/MaxDac/marknotes-web/pull/59)) | Every app cycles the heading through none, H1, H2 and H3 and has strikethrough (`~~`) and a numbered-list toggle. Note links are inserted with `@` in every app |
| Undo and redo | Buttons and keyboard ([MaxDac/marknotes-android#35](https://github.com/MaxDac/marknotes-android/pull/35)) | Partial: awaiting a Libre pin with undo and redo | Keyboard | Keyboard | Android: top-bar buttons, Ctrl+Z, Ctrl+Y and Ctrl+Shift+Z; session-local history of 200 steps, cleared when sync changes the open note |
| Autosave | Yes (800 ms) | Yes (800 ms) | Yes (500 ms) | Yes (400 ms) | Debounce after the last edit |
| Stable note ids in front-matter | Yes | Yes | Yes | Yes | |
| Note links and `@` picker | Yes | Yes | Yes | Yes | Picker ordering differs, see below |
| Duplicate ids given a new id | When the `@` picker opens | When the `@` picker opens | When the `@` picker opens | When the `@` picker opens or a note is opened | [note-format.md](note-format.md#note-ids-front-matter) leaves the timing to the app; links resolve the same way meanwhile |
| Chord sequence editor | Yes | Yes | Yes | Yes | [note-format.md](note-format.md#chord-sequences); `fixtures/chords/*` |
| Chord sequences opt-in setting (off by default) | Yes | Yes | Yes | Yes | [note-format.md](note-format.md#chord-sequences-setting). App-only, no format change |
| Bass fret 12 marker after the fret | Yes | Yes | Yes | Yes | App-only, no format change |
| Import shared text and documents as Markdown | Yes | Yes | Yes | Yes ([MaxDac/marknotes-web#58](https://github.com/MaxDac/marknotes-web/pull/58)) | Android share sheet: text, HTML, `.txt`/`.md`/`.html`, `.docx`, PDF text, images. Desktop: Import… button, drop or paste ([MaxDac/marknotes-electron#55](https://github.com/MaxDac/marknotes-electron/pull/55)); relative links in pasted HTML are dropped. Web: New → Import document, drop or paste, fully offline; inline `data:` images become attachments and PDF import is text only. Web has no share target, because shared content would reach the server. Imported front matter is kept under a new block. App-only, no format change |
| Themes and accent colours | 8 + Dynamic | 8 + Dynamic | 8 | 9 | Light, dark or system everywhere. Dynamic (Material You) needs Android 12. Web adds Pink and defaults to Violet |

### Images

| Feature | Libre | Pro | Desktop | Web | Notes |
| --- | --- | --- | --- | --- | --- |
| Images from file picker / toolbar | Yes | Yes | Yes | Yes | Android also takes a photo with the camera |
| Image paste and drag-and-drop | Yes ([MaxDac/marknotes-android#40](https://github.com/MaxDac/marknotes-android/pull/40)) | Partial: awaiting a Libre pin with the feature | Yes | Yes | Android: "Paste image", Ctrl+V/Shift+Insert, keyboard-app images and drag-and-drop from another app, through the same image import as the picker |
| Unused attachment clean-up | Yes | Yes | Yes | Yes | |
| Remote images blocked | Yes | Partial: awaiting the Libre attachment-only preview pin | Yes | Yes | See known divergence 35 |
| Load remote images on request | Never (by design) | Never (by design) | Per note, `https:` only, for the session | Never (by design) | Android and Web document "never loads remote images" as a privacy property |

### Sync

| Feature | Libre | Pro | Desktop | Web | Notes |
| --- | --- | --- | --- | --- | --- |
| Dropbox sync | Yes | Yes | Yes | Yes | |
| OneDrive sync | No (Pro only) | Yes | Yes | Yes | Every app links several drives, each mirroring the same notes |
| Proton Drive sync | No (Pro only) | Yes | Yes | No | Web: deferred to [MaxDac/marknotes-web#1](https://github.com/MaxDac/marknotes-web/issues/1). Pro and Desktop: folder typed by name; no CAPTCHA, two-password or FIDO2-only accounts. Pro: 64-bit devices only |
| Several Proton Drive accounts | No (Pro only) | Yes ([MaxDac/MarkNotes#41](https://github.com/MaxDac/MarkNotes/pull/41)) | Yes | No | Pro gives every sign-in a new drive id, so the same Proton user can be added as two drives; Desktop keys Proton drives by user id |
| Shared sync algorithm ([marknotes-sync](https://github.com/MaxDac/marknotes-sync)) | Yes | Yes | Yes | Yes | [note-format.md](note-format.md#sync); `fixtures/sync/*` |
| Sync manifest (`.marknotes/manifest.json`) | Yes | Yes | Yes | Yes | |
| Three-way merge, newest wins on conflict | Yes | Yes | Yes | Yes | The other version is saved as a conflicted copy |
| Safe sync mutations and crash recovery (sync 0.3 contract) | Yes (0.3 source) | Yes (0.3 source) | Yes (0.3 source) | Yes (0.3 source) | Source-pinned integrations, not an assumed published release. Unsupported safe mutations fail explicitly; see resolved divergences 32-34 and [Folder deletion](#folder-deletion) |
| Merge a conflicted copy back, with conflict choices | Yes | Yes | Yes | Yes | |
| When sync runs | Launch, Sync now, drive added, after a merge | Same as Libre | Launch or unlock, Sync now, drive added, after a merge | Page load and Sync now | No app polls in the background |
| Sync when the network comes back (opt-in, app open) | Yes ([MaxDac/marknotes-android#38](https://github.com/MaxDac/marknotes-android/pull/38)) | Partial: awaiting a Libre pin with the setting | Yes ([MaxDac/marknotes-electron#51](https://github.com/MaxDac/marknotes-electron/pull/51)) | Yes | "Sync when reconnected", off by default. Web: only while the tab is visible and unlocked. Android: only in the foreground and unlocked. Desktop: while open and unlocked, waiting for an open conflict merge. All sync 2 s after reconnecting and retry after 5, 15 and 30 s on temporary failures; Desktop stops the whole schedule on any permanent failure, Web per drive |
| Sync status | Progress, last sync per drive | Same as Libre | Last sync per drive, "Syncing…" | Last sync per drive ([MaxDac/marknotes-web#53](https://github.com/MaxDac/marknotes-web/pull/53)) | Web records the time only after an error-free pass; Desktop updates it after every attempt |
| Expired sign-in | Reconnect action ([MaxDac/marknotes-android#36](https://github.com/MaxDac/marknotes-android/pull/36)) | Partial: OneDrive and Proton Drive still need Reconnect connectors; remove and re-add | Reconnect action ([MaxDac/marknotes-electron#50](https://github.com/MaxDac/marknotes-electron/pull/50)) | Reconnect action | Reconnect replaces credentials in place, keeping the drive, folder and sync records, and refuses a different account. Android and Desktop match the provider's immutable account id |
| Disconnect signs out of the provider | Yes | Yes (OneDrive: local sign-out) | Yes (OneDrive: local sign-out) | Yes (OneDrive: local sign-out) ([MaxDac/marknotes-web#56](https://github.com/MaxDac/marknotes-web/pull/56)) | Microsoft offers public clients no single-session revoke |
| Case-only renames on the drive | Explicit error | OneDrive only ([MaxDac/MarkNotes#44](https://github.com/MaxDac/MarkNotes/pull/44)) | OneDrive only | OneDrive only | Dropbox and Proton Drive have no safe case-only move |
| Attachment hash check on download | Yes | Yes | Yes | Yes | [note-format.md](note-format.md#attachment-sync) |

### Privacy, diagnostics and app information

| Feature | Libre | Pro | Desktop | Web | Notes |
| --- | --- | --- | --- | --- | --- |
| App lock with idle timeout | Yes | Yes | Yes | Yes (passphrase mode) | See [Local data and privacy](#local-data-and-privacy) |
| Local encryption at rest | Device encryption | Device encryption | Yes | Passphrase mode only | |
| Delete local data | Yes | Yes | Yes | Yes ("Forget this device") | |
| Block screenshots and hide in recent apps | Yes | Yes | Partial: Windows and macOS, best-effort ([MaxDac/marknotes-electron#53](https://github.com/MaxDac/marknotes-electron/pull/53)) | N/A | Browsers offer no such API. Desktop: Settings → Privacy → "Protect notes from screen capture", forced on with the app lock; macOS ScreenCaptureKit can bypass it and Linux has no support |
| Memory-only notes | Not offered | Not offered | Not offered (credentials only, without a keyring) | Yes | Web: notes, images, sync state and credentials stay in memory, single tab. Meant for shared browsers; installed apps keep notes on the device |
| Only the permissions the app needs | Yes (checked on release) | Partial: the `libre/` pin builds marknotes-sync without the AAR permission fix [MaxDac/MarkNotes#38](https://github.com/MaxDac/MarkNotes/issues/38) | N/A | N/A | |
| Local error diagnostics and user-initiated GitHub reporting | Yes | Yes | Yes ([MaxDac/marknotes-electron#54](https://github.com/MaxDac/marknotes-electron/pull/54)) | Yes ([MaxDac/marknotes-web#54](https://github.com/MaxDac/marknotes-web/pull/54)) | See [Android diagnostics](#android-diagnostics). Desktop and Web: Settings › Local diagnostics with "Report a problem", export and Clear logs. Desktop also records renderer process exits; its log holds only times and codes and is not covered by the app lock. Web logs follow the storage mode and start only once storage is unlocked. App-only, no format change |
| About | Licence notices, Ko-fi link | Same as Libre, plus the Proton Drive notice | Short text | No About view; `/security` page and `licenses.txt` | No app shows its version |
| Links to the other apps (Settings and README) | Yes | Yes | Yes | Yes | See [Links between apps](#links-between-apps). App-only, no format change |
| Updates | F-Droid or GitHub releases | GitHub releases | In-app updater | Service worker update | |

## Links between apps

App behaviour only; none of it changes the note format, the manifest or the fixtures.

Settings → About in every app (the Settings dialog on Web), and every app's README, list the other MarkNotes apps so
users can find them. Android Libre and Pro are listed separately, so each app shows exactly three entries and never
links to itself. Android Libre and Pro share the list; each host app tells the shared code which edition it is.

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
  drive is disconnected ("Also delete the notes stored on this device") and in Settings. The deletion never
  propagates to any drive.
  - Android: Settings → Privacy → "Delete all local data" disconnects every drive first. It also clears diagnostics
    and connector data (Pro: the Proton Drive cache and the Microsoft accounts); theme and device id are kept.
  - Desktop: Settings → Privacy → "Delete all local data" disconnects every drive first, rotates the data key and
    keeps app settings. The lock screen also offers a reset for a forgotten passphrase.
  - Web: "Forget this device" (Settings and unlock screen) deletes IndexedDB, local and session storage, the cache
    and the service worker, including the stored drive connections, without signing out of the providers.
- **Remote images**: only note attachments render automatically; `http:`, protocol-relative (`//host`) and `file:`
  images are never loaded. Android and Web never load `https:` images either. Desktop loads them only when the user
  asks, for that note and session.
- **App lock**: optional and off by default.
  - Android: biometric or device credential (requires a screen lock). Locks after Immediately, 1, 5 (default) or 15
    minutes, when the screen turns off and with "Lock now". Turning it on also blocks screenshots.
  - Desktop: passphrase of at least 8 characters (scrypt). Locks after Off, 5, 15 (default) or 60 minutes, on system
    suspend or screen lock and with "Lock now". Turning it on also enables screen-capture protection on Windows and
    macOS.
  - Web: passphrase of at least 10 characters, in passphrase storage mode only. Locks after Off, 5, 15 (default) or
    60 minutes and with "Lock now"; browsers do not report suspend or screen lock.
- **Encryption at rest**: Desktop always encrypts the local note database with a data key protected by the OS keychain,
  or by the app-lock passphrase when set. On Linux without a keyring it warns and keeps that key unprotected, so the
  data is effectively unencrypted unless the app lock is on. Web
  encrypts every local record, paths included, only in passphrase mode, with a key derived from the passphrase
  (PBKDF2-SHA256, 600,000 iterations). Android adds no app-level cipher: with minSdk 29, file-based encryption is
  mandatory, so notes in app-private storage are already encrypted under the device credential and sandboxed from
  other apps.

### Android diagnostics

MarkNotes Libre and Pro share this behaviour, and Desktop
([MaxDac/marknotes-electron#54](https://github.com/MaxDac/marknotes-electron/pull/54)) and Web
([MaxDac/marknotes-web#54](https://github.com/MaxDac/marknotes-web/pull/54)) follow the same rules. It does not change
the note or sync format or add fixture requirements.

- **Local logging** is on by default for Java/Kotlin crashes and handled errors. Native crashes and ANRs are
  excluded initially. Retention is capped at seven days and 5 MiB total, including export snapshots, with pruning
  on startup, write and export; deletion is not guaranteed while the app is force-stopped.
- **Sanitization** uses allowlisted error codes and failure categories only, excluding all user content, paths,
  identifiers, credentials and raw error messages from stored records, draft excerpts and exports.
- **Clear logs** removes local diagnostics, including export snapshots. "Delete all local data" removes them too.
- **Report an error** primarily opens a GitHub issue draft after explicit consent and review of a sanitized,
  bounded recent-error excerpt. The complete encoded URL is limited to 2,000 ASCII characters; include whole
  records only and indicate truncation when records are omitted. The user submits the issue in the browser.
  Before opening, warn that opening the draft already transmits the URL's logs to GitHub, before submission,
  and can leave them in browser history. Libre targets
  [MaxDac/marknotes-android](https://github.com/MaxDac/marknotes-android/issues/new); Pro targets
  [MaxDac/MarkNotes](https://github.com/MaxDac/MarkNotes/issues/new).
- **Share logs** is a secondary, offline local export of the full retained sanitized logs, subject to the same
  retention and total storage limits. Nothing is sent automatically.

## Folder deletion

A folder deleted locally is removed from a drive only when the adapter supports atomic empty-only removal.
Otherwise the agreed behaviour is to retain the remote folder and its per-drive deletion history and offer one
durable manual-removal notice per deletion lifecycle, not an error on every sync. Pending notices survive restart;
acknowledged notices are not issued again. Unchanged retained folders do not reappear locally on that device.
Changed or new descendants, local recreation and replacement identities still reconcile normally; genuine runtime
and persistence failures remain errors, and recursive best-effort deletion is never allowed.
See [deferred drive folder removal](note-format.md#deferred-drive-folder-removal) and
[#50](https://github.com/MaxDac/marknotes-spec/issues/50). Desktop
([MaxDac/marknotes-electron#56](https://github.com/MaxDac/marknotes-electron/pull/56)), Web
([MaxDac/marknotes-web#60](https://github.com/MaxDac/marknotes-web/pull/60)) and Android Libre
([MaxDac/marknotes-android#39](https://github.com/MaxDac/marknotes-android/pull/39)) implement it with
marknotes-sync 0.4.0; Pro declares the capability for OneDrive and Proton Drive when it pins that Libre. Remaining
gaps are tracked below.

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
    creation fails explicitly and retains the affected records instead of adopting or replacing an entry.
    Unsupported empty-only removal now has the intentional deferred-notice contract in item 38
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
35. **`file:` images on Android** (not in fixtures): resolved in Libre by
    [MaxDac/marknotes-android#34](https://github.com/MaxDac/marknotes-android/pull/34): the preview resolves only
    canonical `attachments/<sha256>.<ext>` destinations under the attachment root and renders nothing else. Pro still
    needs a Libre pin containing the fix; its checked pin `d0203b9` renders an explicit `file:` image destination
    outside `attachments/`.
36. Resolved: Web's note tree, breadcrumb, prompts and rename dialog show the title without `.md`, and a rename keeps
    the note's own extension unless one is typed
    ([MaxDac/marknotes-web#55](https://github.com/MaxDac/marknotes-web/pull/55)).
37. **Moving to a reserved name on Android** (not in fixtures): resolved in Libre by
    [MaxDac/marknotes-android#33](https://github.com/MaxDac/marknotes-android/pull/33), with a real-store regression
    checking the error and unchanged source contents. Pro still needs a Libre pin containing the fix; its checked
    pin `d0203b9` can still move a nested `attachments` folder to the root, hiding it from the note tree.
38. **Deferred remote folder removal on Android Pro** (not in fixtures): Libre implements the contract with
    marknotes-sync 0.4.0 ([MaxDac/marknotes-android#39](https://github.com/MaxDac/marknotes-android/pull/39)) and
    passes every `sync/plan` and `sync/executor-safety` case against its own executor
    ([MaxDac/marknotes-android#41](https://github.com/MaxDac/marknotes-android/pull/41)), like Desktop
    ([MaxDac/marknotes-electron#56](https://github.com/MaxDac/marknotes-electron/pull/56)) and Web
    ([MaxDac/marknotes-web#60](https://github.com/MaxDac/marknotes-web/pull/60)). Pro still needs a Libre pin
    containing the contract and must declare `remoteFolderDeletion: unsupported` on its OneDrive and Proton Drive
    stores ([#50](https://github.com/MaxDac/marknotes-spec/issues/50)).

Shared algorithm fixtures (`fixtures/sync/*`, `manifest/*`, `merge/*`, except `sync/executor-safety.json`) are run by
marknotes-sync for every app. App runners may map those algorithms to the library, but must still map every file.
`sync/executor-safety.json` describes executor races and durable restart boundaries that planner inputs cannot
express. marknotes-sync runs it against its reference executor, and Desktop and Web run it against their own
executors, reporting observed store state rather than returning the expected object. Android runs it against its own
executor too ([MaxDac/marknotes-android#37](https://github.com/MaxDac/marknotes-android/pull/37),
[MaxDac/marknotes-android#41](https://github.com/MaxDac/marknotes-android/pull/41)).
Apps also keep their own SDK-facing contract tests for the [drive adapters](note-format.md#drive-adapter-contract).
