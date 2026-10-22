# MarkNotes note format

This is the canonical on-disk format shared by the MarkNotes apps:
[Android](https://github.com/MaxDac/MarkNotes), [desktop](https://github.com/MaxDac/marknotes-electron) and
[web](https://github.com/MaxDac/marknotes-web). Notes and attachments written by one app must be readable by the
others through the same synced drive folder (Proton Drive, Dropbox, OneDrive).

The machine-checkable parts of this document are covered by the JSON fixtures in [`fixtures/`](fixtures/), which every
app runs in CI. Change this document and the fixtures together.

## Notes and folders

A note is a UTF-8 `.md` file; folders are real directories. The note title is the file name without `.md`, and its
full path (for example `Work/Projects/Roadmap`) is the folder path plus the title.

## Note ids (front-matter)

Each note has a stable id stored in a YAML front-matter block at the very top of the file:

```markdown
---
id: k7f3q2xa
---
Note body…
```

- The id has 8 characters, chosen at random from the lowercase Crockford base32 alphabet
  `0123456789abcdefghjkmnpqrstvwxyz` (no `i`, `l`, `o` or `u`). A value that does not match this pattern is not a
  valid id.
- New notes get an id when they are created. A note without an id (a legacy note, or one written by another tool) gets
  one lazily: when it is opened in the editor, or when the `@` popup needs to list it. The id is written back through
  the normal save path, so it syncs like any other edit.
- Any other front-matter keys are kept verbatim. When the block has an `id:` line it is replaced; otherwise `id` is
  added as the first key. The block keeps its line endings (CRLF if it uses CRLF).
- The front-matter is hidden in the editor and preview: you edit only the body, and the header is re-attached on save.
- The id stays the same when the note is renamed or moved, so links keep working.
- **Duplicate ids** (for example when a file is copied): links resolve to the first note by sorted path (code-point
  order). When an app notices a duplicate locally, it gives the later note a new id. A `(conflicted copy)` note gets
  a new id too, so links keep pointing at the original.

## Note links

A link to another note uses the `marknotes:` scheme with the note id:

```markdown
[Project plan](marknotes:k7f3q2xa)
```

When a link is inserted from the `@` popup, the link text is the note's title, with `[`, `]` and `\` escaped with a
backslash; users may edit it freely afterwards. Because links use the id, they survive renames and moves. In the
preview, clicking the link opens the note. If no note has that id, the link is shown as broken and clicking it reports
**Note not found**.

In the editor, typing `@` at the start of a line or after whitespace opens a picker. It lists every note with its
title, full path (`Folder/Subfolder/Title`) and id, which has a copy button. Text typed after `@` filters the list by id
prefix, or by a case-insensitive substring of the title or full path. Arrow keys move, Enter or a click inserts the
link in place of `@query`. Esc closes the picker; so does typing a space when nothing matches.

## Images (attachments)

Images are stored once, at the **root** of the notes folder:

```text
attachments/<sha256>.<ext>
```

- `<sha256>` is the lowercase hex SHA-256 of the final, stored bytes; `<ext>` is `png`, `jpg`, `gif` or `webp`, taken
  from the MIME type. Other image types are rejected.
- Files in `attachments/` whose name does not match `<64 lowercase hex>.<ext>` are not attachments and are ignored.
- Before storing, images are downscaled so that the longest side is at most 2048 px, keeping their format. GIFs are
  stored untouched. Images larger than 10 MB after processing are rejected with a message.
- The inserted markdown is `![<original file name without extension>](attachments/<sha256>.<ext>)`. The path is
  always resolved from the notes root, whatever folder the note is in.
- Images can be added with an **Insert image** action, by pasting, or by drag-and-drop where the platform allows it.
- The `attachments/` folder is hidden from the note tree, and `attachments` cannot be used as a root note or folder
  name (on create or rename).
- **Settings → Clean unused images** scans every note for `attachments/…` references and deletes unreferenced
  attachment files locally; the deletion reaches the drives on the next sync. Cleanup never runs automatically.

## Sync manifest

Each linked drive has `.marknotes/manifest.json` in the synced folder, rewritten after each sync:

```json
{"format":1,"app":"MarkNotes","deviceId":"…","updatedAt":1700000000000,"files":{"Work/Roadmap.md":{"sha256":"…","version":"…","syncedAt":1700000000000}}}
```

- Compact JSON, keys in the order shown, `files` sorted by path, entry keys `sha256`, `version`, `syncedAt`.
- Readers reject a manifest whose `format` is greater than 1, or that lacks `deviceId`, `updatedAt` or any entry
  field; unknown keys are ignored.

## Three-way merge

When a note changed both locally and on a drive, apps run a line-based diff3 merge against the last synced base
(the web app does not merge yet; see [features.md](features.md)).
Unresolved conflicts are written with the markers `<<<<<<< this device`, `=======` and `>>>>>>> drive`; a conflict can
be resolved as local, remote, both (local then remote) or neither. When a merge is not possible the remote version is
kept as `<name> (conflicted copy).md` next to the note.

## Attachment sync

Files under `attachments/` sync to **every** linked drive, like notes, and are listed in the manifest with their
`sha256` (the manifest stays `format: 1`). They are binary and immutable, because the name is the content hash:

- they are only uploaded, downloaded or deleted, never three-way merged, and they never produce conflicted copies;
- no base copy is kept for them;
- a download whose content does not match the hash in its name is rejected;
- deletions propagate in both directions, like notes.
