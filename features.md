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
| Stable note ids in front-matter | Yes | Yes | Yes | |
| Note links and `@` picker | Yes | Yes | Yes | Picker ordering differs, see below |
| Images from file picker / toolbar | Yes | Yes | Yes | |
| Image paste and drag-and-drop | Not verified | Yes | Yes | |
| Unused attachment clean-up | Yes | Yes | Yes | |
| Google Drive sync | Yes | Yes | Yes | |
| Proton Drive sync | Yes | Yes | No | |
| Sync manifest (`.marknotes/manifest.json`) | Yes | Yes | Partial | Web: only `format` and `files[].sha256` verified |
| Three-way merge with conflict choices | Yes | Yes | Not verified | Android and Desktop share the same test vectors |
| Automatic sync | At launch and manual | Background polling | Background polling | |
| Trash | Yes | Yes | No | |
| Attachment hash check on download | Not verified | Not verified | Yes | |

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
11. **Hash check on download**: only Web is known to reject attachments whose bytes do not match their name.
12. **Duplicate ids**: Android compares ids case-insensitively; Desktop and Web compare them exactly.
13. **Manifest and merge on Web**: not yet covered by Web tests; the Web parity runner is expected to confirm or report
    differences.
