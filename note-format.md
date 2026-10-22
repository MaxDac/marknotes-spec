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

A note whose front-matter has `type: chords` is a [chord sequence](#chord-sequences) and opens in the chord editor.
It is still a note in every other respect.

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

## Chord sequences

A chord sequence is a note that stores guitar and bass chord shapes instead of free text, ported from
[ChordBuilder](https://github.com/MaxDac/ChordBuilder). It is a normal UTF-8 `.md` note: ids, links, rename, move,
sync, the manifest, three-way merge and conflicted copies work exactly as for any other note. One file holds one
ordered sequence (a song or progression); the order in the file is the display order.

```markdown
---
id: k7f3q2xa
type: chords
---
# Autumn Leaves

## Am7 (guitar)
frets: x 0 2/2 0 1/1 0
tags: jazz, open

First chord of the A section.
Let it ring.

## Am7 (bass)
frets: 5/1 7/3 7/4 x
```

### Detection

A note is a chord sequence when its front-matter block (as in [Note ids](#note-ids-front-matter): opening fence on the
first line, trailing spaces or tabs allowed, and a closing `---` line) contains a `type` line whose value is `chords`:

- The key is `type` at the start of the line, followed by `:` and at least one space or tab. Indented lines,
  `Type:` and `type:chords` do not count.
- The value is the rest of the line without surrounding spaces and tabs and, if it is wrapped in one pair of matching
  `"` or `'`, without them. It must then equal `chords` exactly (case-sensitive). YAML comments are not supported:
  `type: chords # song` is not a chord sequence.
- Only the first `type` line of the block counts.

Opening a chord sequence shows the chord editor. If its body does not [parse](#parsing), the app opens the note in the
normal note editor instead and shows the parse error with its line number, so nothing is lost. A new chord sequence is
created with the front-matter `---\nid: <new id>\ntype: chords\n---\n` and an empty body. Other front-matter keys are
kept verbatim, like for any note.

### Model

A sequence has an optional `title` (`null` when absent) and an ordered list of `chords`. Each chord has:

| Field | Value |
| --- | --- |
| `instrument` | `guitar` (6 strings, tuning E2 A2 D3 G3 B3 E4) or `bass` (4 strings, tuning E1 A1 D2 G2) |
| `baseName` | Chord name, for example `Am7` |
| `strings` | One entry per string, lowest string first (see below) |
| `tags` | List of tags |
| `note` | Free text, `""` when there is none |

A string is `{"state":"muted"}`, `{"state":"open"}` or `{"state":"fretted","fret":<n>,"finger":<n>}`.

Whitespace below always means the ASCII characters U+0009 to U+000D and U+0020 only (a no-break space is not
whitespace). Lengths are counted in Unicode code points. Lowercasing is Unicode default, locale-independent
lowercasing (`toLowerCase()` in JavaScript, `lowercase()` in Kotlin, `String.downcase/1` in Elixir).

Normalisation, applied by the parser, the serializer and the validator:

- `title`: trimmed; an empty title is the same as no title.
- `baseName`: trimmed, and every run of whitespace replaced by one space.
- `note`: `\r\n` converted to `\n`, then trimmed. Inner lines are kept verbatim.
- `tags`: each tag trimmed, empty tags dropped, and duplicates removed by comparing lowercased tags, keeping the first
  spelling and the original order.

### Body format

The body is the text after the front-matter block. Lines are separated by `\n`; one `\r` at the end of a line is
removed, so CRLF files parse the same. Line numbers are 1-based and count from the first line of the body.

- **Title** (optional): the first non-blank line may be `# <title>`.
- **Chord heading**: `## <baseName> (<instrument>)`. Every line starting with `##` and a space is a chord heading.
  The instrument is the last `(guitar)` or `(bass)` on the line (lowercase), optionally followed by spaces or tabs;
  everything between `##` plus its space and the instrument is the name, so `## C (add9) (guitar)` is named
  `C (add9)` and `## Am7(bass)` is accepted. Sequence numbers are not written in the file; they are
  [derived](#display-names).
- **Frets**: the next non-blank line after the heading must start with `frets:`. The rest of the line is split on
  runs of spaces and tabs into one token per string, lowest string first: `x` (muted), `0` (open) or
  `<fret>/<finger>` (fretted), where `<fret>` is one or two digits and `<finger>` one digit. An empty `frets:` line
  gives no strings.
- **Tags** (optional): if the next non-blank line after `frets:` starts with `tags:`, the rest of the line is split on
  `,` into tags. An empty `tags:` line gives no tags.
- **Note**: every remaining line up to the next chord heading or the end of the body, joined with `\n` and trimmed.
  Markdown, blank lines, `# …` lines and further `frets:` or `tags:` lines are note text.

Keys (`frets:`, `tags:`) and instruments are case-sensitive. Blank lines may appear anywhere.

#### Parsing

Parsing returns either the normalised sequence or the first error, with its line number and one of these messages:

| Situation | Line | Message |
| --- | --- | --- |
| A non-blank line before the first chord that is not the title | That line | `Expected a chord heading.` |
| A line starting with `##` and a space, without `(guitar)` or `(bass)` at the end | That line | `Invalid chord heading.` |
| No `frets:` line after a heading | The heading | `Chord has no frets line.` |
| A frets token that is not `x`, `0` or `<fret>/<finger>` | The frets line | `Invalid string "<token>".` |

Parsing checks only the syntax; out-of-range numbers, wrong string counts and empty names are reported by
[validation](#validation).

#### Serialization

The chord editor writes the body below and keeps the front-matter block unchanged. All apps must produce the same
bytes. Values are normalised first.

1. If there is a title: `# <title>`.
2. For each chord, after one blank line if anything was written before it:
   1. `## <baseName> (<instrument>)`;
   2. `frets:`, then each token preceded by one space (`x`, `0` or `<fret>/<finger>`, without leading zeros);
   3. if there are tags, `tags:`, one space and the tags joined with a comma and a space (`a, b`). Without tags
      the line is omitted, except that a bare `tags:` line is written when the note's first line starts with
      `tags:`, so that it is not read back as tags;
   4. if the note is not empty, one blank line and then the note lines.
3. Every line, including the last, ends with the line ending. An empty sequence is the empty string.

The line ending is `\r\n` when the note's front-matter block uses CRLF (its opening fence line ends with `\r\n`), and
`\n` otherwise. Serializing a parsed canonical body gives back the same bytes. Apps write the file only when the user
changes something, so opening a chord sequence never creates a sync change.

### Validation

The chord editor validates the normalised sequence before saving and shows every error. Errors are listed in this
order: the title, then each chord in file order and, within a chord, in the order of the table. Each message appears
at most once per chord and carries the chord's 0-based index (`null` for the title).

| Check | Message |
| --- | --- |
| The title contains a line break | `Title must be a single line.` |
| `baseName` is empty | `Enter a chord name.` |
| `baseName` is longer than 80 | `Chord names must be at most 80 characters.` |
| The number of strings is not the instrument's | `guitar requires 6 strings.` or `bass requires 4 strings.` |
| Fewer than three strings are not muted | `Select at least three non-muted strings.` |
| A fret is outside 1-24 | `Frets must be between 1 and 24.` |
| A finger is outside 1-4 | `Fingers must be between 1 and 4.` |
| `note` is longer than 1000 | `Notes must be at most 1000 characters.` |
| A note line starts with `##` and a space | `Note lines cannot start with "## ".` |
| There are more than 50 tags | `Use at most 50 tags.` |
| A tag is longer than 80 | `Tags must be at most 80 characters.` |
| A tag contains `,`, `\n` or `\r` | `Tags cannot contain commas or line breaks.` |

An empty sequence is valid.

### Display names

Each chord is shown as `<sequence> <baseName>`. The sequence numbers the chords that have the same instrument and the
same lowercased `baseName` 1, 2, 3, … in file order: the second guitar `Am7` is `2 Am7`, while the first bass `Am7` is
`1 Am7`. Each chord shows its own spelling of the name.
