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
| Stable note ids in front-matter | Yes | Yes | Yes | |
| Note links and `@` picker | Yes | Yes | Yes | Picker ordering differs, see below |
| Images from file picker / toolbar | Yes | Yes | Yes | |
| Image paste and drag-and-drop | No | Yes | Yes | Android: file picker and camera only |
| Unused attachment clean-up | Yes | Yes | Yes | |
| Dropbox sync | Yes | Yes | Yes | |
| OneDrive sync | Yes | Yes | Yes | |
| Proton Drive sync | Yes | Yes | No | Web: deferred to marknotes-web#1 |
| Shared sync algorithm ([marknotes-sync](https://github.com/MaxDac/marknotes-sync)) | Yes | Yes | Yes | [note-format.md](note-format.md#sync); `fixtures/sync/*` |
| Sync manifest (`.marknotes/manifest.json`) | Yes | Yes | Yes | |
| Three-way merge, newest wins on conflict | Yes | Yes | Yes | The other version is saved as a conflicted copy |
| Merge a conflicted copy back, with conflict choices | Yes | Yes | Yes | |
| When sync runs | Launch and Sync now | Launch and Sync now | Page load and Sync now | No app polls in the background |
| Attachment hash check on download | No | No | Yes | Required by [note-format.md](note-format.md#attachment-sync); see divergence 11 |

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
11. **Hash check on download** ([#12](https://github.com/MaxDac/marknotes-spec/issues/12)): Android and Desktop
    write and record attachments whose bytes do not match their name; they must report a step error instead. Only Web
    implements the check.
12. **Duplicate ids**: Android compares ids case-insensitively; Desktop and Web compare them exactly.
13. Resolved: manifest parsing on Web now uses marknotes-sync.
14. **MIME case** (`attachments/extension-for-type`): Web matches MIME types case-sensitively.
15. Resolved: manifest parsing on Android now uses marknotes-sync.
16. **Mention detection on Android** (`note-links/mention`): Android opens the picker for `@` followed by a space and
    closes it only when the query has no results.
17. **Missing function on Web**: no note id validator (`note-ids/validation`); the Web parity runner skips that file.
    The three-way merge now comes from marknotes-sync.
18. Resolved: all three apps implement chord sequences and run `fixtures/chords/*`.
19. **Re-attaching front-matter** (`front-matter/with-body`,
    [#21](https://github.com/MaxDac/marknotes-spec/issues/21)): all three apps concatenate the header and the body, so
    a header without a final line break gives `---Hello`.
20. **Compare before write** ([#9](https://github.com/MaxDac/marknotes-spec/issues/9)): no app re-hashes a local file
    before writing or deleting it during a sync, and uploads record the hash from the listing.
21. **Conditional deletes** ([#10](https://github.com/MaxDac/marknotes-spec/issues/10)): Android and Desktop delete
    Dropbox and OneDrive files unconditionally; so does Web on Dropbox.
22. **Modification time** ([#11](https://github.com/MaxDac/marknotes-spec/issues/11)): Android and Desktop read the
    server times (Dropbox `server_modified`, OneDrive `lastModifiedDateTime`) and use the current time for Proton
    Drive uploads and for downloaded files. Web reads the client fields; its download times are not verified.
23. **Local folder creation** ([#15](https://github.com/MaxDac/marknotes-spec/issues/15)): Android and Desktop do not
    count it as a local change when queueing drives again.
24. **Complete listings** ([#16](https://github.com/MaxDac/marknotes-spec/issues/16),
    [#17](https://github.com/MaxDac/marknotes-spec/issues/17)): on Android and Desktop a failed OneDrive full listing
    leaves its delta checkpoint behind, and Proton Drive entries whose name cannot be decrypted are treated as absent.
25. **Conflict order** ([#18](https://github.com/MaxDac/marknotes-spec/issues/18)): all three apps overwrite the
    losing version before saving its conflicted copy.
26. **Pending conflicts** ([#22](https://github.com/MaxDac/marknotes-spec/issues/22)): Android and Desktop keep them
    in memory only, so they are forgotten on restart.
27. **Media types** ([MaxDac/marknotes-electron#31](https://github.com/MaxDac/marknotes-electron/issues/31)): Desktop
    uploads attachments to Proton Drive as `text/markdown`.

Sync fixtures (`fixtures/sync/*`, `manifest/*`, `merge/*`) are run by marknotes-sync, which implements them for every
app. App runners may map them to the library instead of running them again, but must still map the files. Apps keep
their own contract tests for the [drive adapters](note-format.md#drive-adapter-contract), which no fixture reaches.
