# MarkNotes note format

This is the canonical on-disk format shared by the MarkNotes apps:
[Android](https://github.com/MaxDac/marknotes-android) (including [Pro](https://github.com/MaxDac/MarkNotes)),
[desktop](https://github.com/MaxDac/marknotes-electron) and
[web](https://github.com/MaxDac/marknotes-web). Notes and attachments written by one app must be readable by the
others through the same synced drive folder (Proton Drive, Dropbox, OneDrive).

The machine-checkable parts of this document are covered by the JSON fixtures in [`fixtures/`](fixtures/), which every
app runs in CI. Change this document and the fixtures together.

## Notes and folders

A note is a UTF-8 `.md` file; folders are real directories. The note title is the file name without `.md`, and its
full path (for example `Work/Projects/Roadmap`) is the folder path plus the title.

A note whose front-matter has `type: chords` is a [chord sequence](#chord-sequences) and opens in the chord editor
when the [Chord sequences setting](#chord-sequences-setting) is on. It is still a note in every other respect.

## Note ids (front-matter)

Each note has a stable id stored in a YAML front-matter block at the very top of the file (after a UTF-8 byte order
mark, if the file starts with one):

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
- A byte order mark before the opening fence belongs to the block: it is hidden with it and stays first. When a note
  that starts with a byte order mark gets a new block, the mark stays before it.
- The front-matter is hidden in the editor and preview: you edit only the body, and the header is re-attached on save.
  When the header does not end with a line break (its closing `---` is the last line of the file) and the body is not
  empty, a line break is inserted between them: `\r\n` if the block's opening fence line ends with `\r\n`, `\n`
  otherwise.
- The id stays the same when the note is renamed or moved, so links keep working.
- **Duplicate ids** (for example when a file is copied): links resolve to the first note by sorted path (UTF-16
  code-unit order, as for [sync paths](#what-is-synced)). When an app notices a duplicate locally, it gives the later
  note a new id. A `(conflicted copy)` note gets a new id too, so links keep pointing at the original.

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

## Sync

The apps sync the notes folder with each linked drive using **one shared implementation**,
[marknotes-sync](https://github.com/MaxDac/marknotes-sync): a Rust library used through Kotlin bindings on Android
and a WebAssembly npm package on desktop and web. It decides what to do; each app only lists files, transfers them
and stores what it needs. The rules below are what the library implements and the [`fixtures/sync/`](fixtures/sync/)
fixtures check; an app must not add its own sync decisions.

### What is synced

- Notes (`.md`, any case) outside `attachments/`, [attachments](#images-attachments), and every folder.
- Paths are relative, `/`-separated and already normalised: no empty segment, no segment that is `.`, `..`, starts with
  `.`, has leading or trailing whitespace or contains a control character (U+0000 to U+001F, or U+007F), which some
  file systems and drives reject. Anything else (other files, hidden files and folders such as
  `.marknotes/`, notes inside `attachments/`) is ignored on both sides.
- A path that is a file on one side and a folder on the other is reported as the error
  `is a file on one side and a folder on the other` and skipped.
- A path listed more than once in the local tree or on the drive, or recorded more than once, is reported as the
  error `is listed more than once` and skipped together with everything inside it: the listing or the records are
  inconsistent, so nothing is uploaded, downloaded, deleted, recorded or forgotten for it.
- A folder that a listing does not include, although it includes entries inside it (some drives do not list the
  folders that hold files, such as `attachments/`), certainly exists on that side: it gets no folder action (it is
  neither created, recorded, deleted nor forgotten) and its contents are handled one by one.
- Paths are identified case-insensitively, by Unicode simple case folding, because Dropbox and OneDrive ignore case
  (`Über.md` and `über.md` are the same path; `Straße.md` and `STRASSE.md` are not). Synced paths that differ only in
  case, whether on the same side or across the local tree, the drive and the records, are each reported as the error
  `differs only in case from another path` and skipped, together with everything inside them, except for a
  [provably safe case-only rename](#case-only-renames): nothing is uploaded, downloaded, deleted, recorded or forgotten
  for an ambiguous collision. A two-step rename through another name is still supported, but a device that misses
  the intermediate name can converge using the saved record.
- Errors come first in the plan, one per path, by path; when several apply, the file/folder message wins, then
  `is listed more than once`, then the case message.
- Paths are ordered by UTF-16 code units, the order of JavaScript's default string sort.

### Records and base copies

After a path is synced the app saves a **record** for it, per drive: the drive id and version of the file, the
SHA-256 of the local bytes and the time. For notes it also keeps a **base copy**, the text both sides agreed on, used
by the three-way merge. Folders get a record with their drive id. Attachments get no base copy.

A file record can also be **flagged as deleted locally** (`localDeleted`): a sync saw the file missing from the local
tree after the record was saved. After listing the local tree, and before planning, the app asks marknotes-sync
(`local_deletions`, checked by the [`fixtures/sync/local-deletions.json`](fixtures/sync/local-deletions.json)
fixtures) which records of **every** linked drive to flag, not only those of the drive being synced, and saves the
flags. `local_deletions` returns the file records that are not flagged yet and whose path is not a file in the local
listing (exact path match); folder records are never flagged. Every record the app writes for an action is unflagged,
and records saved without the flag read as unflagged.

The flag matters with several drives. Without it, a note moved away and back between syncs could look unchanged to a
drive whose record predates the move, and that drive's copy of the deletion would then delete the restored note
everywhere ([#37](https://github.com/MaxDac/marknotes-spec/issues/37)). Dropping the other drives' records instead
would not work: a file only on the drive and without a record is downloaded, so a local deletion would come back from
every drive but the first.

### Sync plan

Each sync of one drive lists the local tree (hashing every local file), the drive tree and
`.marknotes/manifest.json`, and computes an ordered plan:

1. **Folders**, parents first (by depth, then path):
   - on both sides: recorded if not yet recorded with that drive id;
   - only local: created on the drive if it was never synced, still holds something to sync (including a file whose
     record is flagged as deleted locally), or holds anything that is not synced (an ignored file or folder, such as a
     PDF or `.obsidian/`), otherwise deleted locally;
   - only on the drive: created locally under the same rule, otherwise deleted on the drive;
   - on neither: its record is dropped.

   A folder kept this way is not deleted: its synced contents are handled one by one in step 2, so those deleted on
   the other side are deleted individually. The listings given to the planner therefore include ignored entries.
2. **Files**, by path. A side has *changed* when there is no record, or when the local hash (or the drive id or
   version) differs from the record.
   - On both sides, never synced or changed on both: if the manifest lists this exact drive version with the local
     hash the file is just recorded; otherwise notes are [reconciled](#conflicts) and attachments are recorded (their
     name is their content).
   - On both sides, only one changed: the change is uploaded as a new revision or downloaded. When the drive has a new
     version whose manifest hash equals the recorded local hash (another device re-uploaded the same content), it is
     only recorded.
   - Only local: uploaded if never synced, changed, or its record is flagged as deleted locally (the file was deleted
     and put back here, so it is new to this drive); otherwise it was deleted on the drive, so it is deleted locally.
   - Only on the drive: downloaded if never synced or changed; otherwise it was deleted locally, so it is deleted on
     the drive.
   - So a change wins over a deletion. A file inside a folder to be deleted still gets its own guarded file
     deletion before that folder is removed.
   - In this step the flag only changes the "only local" case above. Elsewhere a flagged record is compared as usual,
     and a file on both sides that would get no action is recorded again (no upload, no reconcile) to clear the flag.
   - On neither side: its record is dropped.
3. **Folder deletions**: a folder deleted on one side, with nothing changed under it on the other, is removed.
   Folder deletions run deepest-first (then by path), after the guarded deletions of their known files.
   Each local folder removal is non-recursive and fails if any entry remains. Each drive folder removal
   must be an atomic empty-only operation: it must not remove any remaining descendant, including an ignored entry
   or a file another device created after the listing. A fresh listing followed by an ordinary recursive deletion
   does not provide this guarantee. If the folder is not empty, or the provider cannot guarantee empty-only removal,
   the step fails explicitly without a recursive fallback.

   Records are dropped only after their corresponding deletion succeeds. A failed folder removal retains its
   folder record and any records not already successfully deleted, so a later sync can retry or pick up new
   contents instead of recreating the folder from a failed deletion.

A failed step is reported as `<path>: <message>` and the sync goes on with the next one.

### Applying the plan

- Apps flush pending editor saves before a sync starts.
- Before every local write or deletion (a download, a merge or conflict result, a local file or folder deletion), the
  app hashes the local file again and skips the step if its hash differs from the listing the plan was computed
  from: the file was edited during the sync. A new local file must not be overwritten.
  Folder removal is non-recursive and empty-only, so
  neither a new file nor an ignored descendant is deleted. A skipped changed-file step is not an error and records
  nothing; the next sync picks the edit up.
- An upload records the hash of the bytes actually uploaded, not the hash from the listing.
- Creating a new drive file (`uploadNew`), a conflicted copy or a drive folder (`createRemoteFolder`) is
  **create-only**. If the path became occupied after the listing, the action fails without replacing or automatically
  renaming the existing entry, and without saving a successful record for the attempted creation. The next sync
  takes a fresh listing and reconciles the two sides. Creating an internal/root directory idempotently is a
  separate adapter operation, not a substitute for a planned create-only action.
- A new local conflicted copy is created atomically only if its path is absent. A name selected from a listing
  is not a reservation and does not permit overwriting a file created since that listing.

### Case-only renames

A saved record can establish which spelling is stale when there is exactly one local entry, one drive entry and
one consistent recorded spelling for a case-folded path, and both entries have the same kind:

- If the record has the local spelling and the local file hash still matches the record, rename the local entry to
  the drive spelling.
- If the record has the drive spelling and the drive id and version still match the record, rename the drive entry
  to the local spelling.
- For a folder, all descendants must map one-to-one by the same prefix change, with records proving that every
  stale-side file is unchanged. Missing, ignored, duplicate, differently typed or ambiguously cased descendants
  prevent automatic inference. Genuine same-side collisions, absent or inconsistent records, and an edited stale
  side keep the collision errors.

Renames do not change content, note ids or modification times. They use no-overwrite operations and recheck the
stale-side identity/content before changing it. A failed precondition or an unsupported safe rename is an explicit
step error that keeps the records. A local file system may use a private temporary name to implement a case-only
rename, but must make an interrupted rename recoverable and must not overwrite another entry at either step.
Do not emulate a drive rename with an unannounced temporary path: another device could interpret the missing
original as a deletion between the two moves.

After a successful rename, migrate the current drive's affected record and base-copy paths together. Keep the
other drives' records and bases at their previous paths until those drives perform their own convergence; their
saved identity and version remain evidence of the unchanged stale drive spelling. A `localDeleted` flag is not
permission to delete a case alias. Do not apply additional actions inside a renamed subtree from the old listing:
take a fresh listing to plan its contents. A local rename counts as a local change for the other drives' queue.
When the rename and history migration cannot commit atomically, persist a per-drive rename intent before the
mutation and replay it before flagging local deletions or planning. A restart after the move but before metadata
migration must finish migrating the records and bases, not leave an old recorded spelling as a permanent collision.
Replay verifies the entry's identity at its source or intended destination and never overwrites an intervening edit.
Migration preserves the recorded hashes and drive versions: a rename response is not proof that its returned
revision contains the already-synced bytes. In particular, a stable-id no-overwrite drive move may preserve a
concurrent content edit; its new revision must still be detected by the next plan, not adopted as synced.

### Modification time

The modification time of a note is the time it was last edited on a device. An app sets it to the current time only
for a real edit or when it writes a merge result; uploads carry it to the drive and downloads set the local file's
modification time to the drive's value (see the [drive adapter contract](#drive-adapter-contract)). Syncing a note to
another device therefore never makes it newer.

### Conflicts

A note that changed on both sides is reconciled:

1. Same text on both sides: it is recorded.
2. A base copy exists and the line-based three-way merge is clean: the merged text is written locally, uploaded and
   recorded.
3. Otherwise **the newest version wins**, by [modification time](#modification-time); on a tie the drive wins. The
   winner is kept at the note's path on both sides. The other version is saved next to it as
   `<name> (conflicted copy).md` (or `<name> (conflicted copy) <n>.md`, with the first free `n` from 2; a name is
   taken when an existing local path equals it by [simple case folding](#what-is-synced)) and synced
   as a new note. The copy always gets a new valid id (set as in [Note ids](#note-ids-front-matter)), even when the
   losing version had no id or an invalid one. Its modification time is the losing version's modification time,
   not the time of the sync or the copy's creation. Creating or retrying the copy and uploading or downloading it
   preserve that time; only a subsequent real edit makes it newer.

   The steps run in this order, so the losing version is never lost: write the copy locally; then overwrite the
   losing side (write the drive's text locally, or upload the local text as a new revision) and record the note;
   then upload the copy. If writing the copy fails, the note is left as it is and the step is reported.

   Before creating the copy, persist a durable intent identifying the drive, original conflict, selected copy path,
   new note id, exact copy text and losing modification time. After a crash, reuse that intent only for the same
   conflict, and reuse an existing copy only after verifying its exact text (including the replacement id) and
   modification time. If the intended path is absent, retry its create-only write with the same id and time.
   Validate persisted intent paths and ids before reuse: the copy must have a conventional conflicted-copy name
   beside the original, never the original path itself or an arbitrary destination, and its replacement id must
   be valid.
   An unrelated or edited file is never accepted as the saved losing version or overwritten: preserve it and
   allocate a new free copy path and intent. A stale intent must not permanently prevent a new conflict from
   being reconciled. Persist the intent before the copy, and verify the copy is durable before overwriting the
   losing side. Keep enough recovery state through winner recording and copy upload to finish an interrupted
   action without manufacturing a duplicate. Clear recovery state only after those steps are durably complete;
   a replay must not overwrite a winner or copy that has since been edited.

Every app tells the user about conflicted copies and offers to merge a copy back into its note: a three-way merge of
the base, the note and the copy, where each conflicting region can be resolved as local, remote, both (local then
remote) or neither. Unresolved regions are written with the markers `<<<<<<< this device`, `=======` and
`>>>>>>> drive`. Pending conflicts (copy path, note path, drive and base text) are kept across restarts until they
are resolved or one of the two files is gone.

The three-way merge always takes this device's text (the note) first and the other text (the drive's version, or the
copy) second, as `local` and `remote` in the `merge/merge3` fixtures. The merged text, the conflicting regions and
their number can depend on that order, notably without a base, so swapping the two is not allowed.

### Sync manifest

Each linked drive has `.marknotes/manifest.json` in the synced folder:

```json
{"format":1,"app":"MarkNotes","deviceId":"…","updatedAt":1700000000000,"files":{"Work/Roadmap.md":{"sha256":"…","version":"…","syncedAt":1700000000000}}}
```

- It lists every synced file (notes and attachments, not folders) with the drive version and the SHA-256 of its
  content, so that a device can tell that a drive version holds content it already has.
- Compact JSON, keys in the order shown, `files` sorted by path, entry keys `sha256`, `version`, `syncedAt`.
- It is rewritten after a sync only when a file's path, hash or version changed.
- Readers reject a manifest whose `format` is not a number up to 1, that lacks `deviceId` (a string) or `updatedAt`,
  whose `files` is not an object, or with an entry lacking a string `sha256` or `version` or a numeric `syncedAt`;
  a missing `files` means no files, and unknown keys are ignored. A rejected manifest is treated as absent.

### Several drives

With more than one drive the app syncs them in order. When a drive brings local changes (downloads, local folder
creations, local file or folder deletions, merges or conflicted copies), every other drive that is not already
waiting is queued again so that it receives them, at most 3 passes per drive. A drive whose sync failed brings no
changes. Before each drive's sync the app flags, in the records of every linked drive, the files missing locally (see
[Records and base copies](#records-and-base-copies)).

### Attachment sync

Attachments sync to **every** linked drive like notes and are listed in the manifest with their `sha256` (the manifest
stays `format: 1`). They are binary and immutable, because the name is the content hash:

- they are only uploaded, downloaded or deleted, never three-way merged, and they never produce conflicted copies;
- no base copy is kept for them;
- a download whose content does not match the hash in its name is a step error: nothing is written or recorded;
- deletions propagate in both directions, like notes.

### Drive adapter contract

Each app talks to each drive through its own adapter, which lists the drive and transfers files for marknotes-sync.
The sync fixtures cannot reach the adapters, so each app keeps contract tests for them against a fake HTTP or SDK
layer. Every adapter must provide:

- **Complete listings.** A listing is either complete or it fails, and a failed listing fails the sync of that drive.
  An entry that cannot be read (for example a Proton Drive name that cannot be decrypted) fails the listing; it is
  never treated as absent, which would read as a deletion. An incremental listing (OneDrive delta) stores its
  checkpoint only after a complete listing succeeds, and clears it when one fails, so that the next sync lists
  everything again.
- **Conditional changes.** Uploads of a new revision and deletions of files only apply to the version in the listing:
  Dropbox `update` mode with the `rev` and `delete_v2` with `parent_rev`, OneDrive `If-Match` with the eTag, and on
  Proton Drive a check of the active revision before trashing. A failed precondition is a step error that keeps the
  record, so the next sync sees the other device's change.
- **Safe folder removal.** Folder deletion must be atomic and empty-only, after individual conditional file
  deletions. Recursive provider deletion, even after an empty listing or a file-revision check, is not equivalent.
  If an adapter cannot provide this guarantee, it reports an unsupported-operation step error and leaves the
  folder and its remaining records intact.
- **Create-only writes.** New files, conflicted copies and planned new folders fail if an entry already exists:
  Dropbox uses `mode: add`, `autorename: false` and `strict_conflict: true`; OneDrive uses
  `@microsoft.graph.conflictBehavior=fail` on the creation operation. Other providers must give the equivalent
  no-replacement guarantee or fail explicitly. A folder conflict must not be converted to success by looking up
  and adopting the existing entry.
- **Case-only rename capabilities.** A drive rename must preserve the stable entry identity and all content,
  fail without overwriting an occupied destination, and leave recorded content versions unchanged until verified
  by a fresh plan. Dropbox's [`move_v2`](https://docs.dropboxapi.com/dropbox-api/api-reference/user-endpoints/files/move-v-2)
  explicitly does not support case-only renames, so an adapter must report that limitation instead of claiming a
  successful direct move. This does not prevent a device from converging its unchanged local spelling to a
  case-only rename it observes on the drive.
- **Modification times.** Uploads write the note's [modification time](#modification-time) to Dropbox
  `client_modified`, OneDrive `fileSystemInfo.lastModifiedDateTime` or the Proton Drive claimed `modificationTime`.
  Listings read the same fields, falling back to the server time when they are missing, and downloads set the local
  file's modification time to them.
- **Media types.** Notes are uploaded as `text/markdown`, attachments with the MIME type of their extension
  (`image/png`, `image/jpeg`, `image/gif` or `image/webp`) and the manifest as `application/json`.

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
first line, after a byte order mark if there is one, trailing spaces or tabs allowed, and a closing `---` line)
contains a `type` line whose value is `chords`:

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

### Chord sequences setting

Chord sequences are opt-in. Each app has a **Settings → Chord sequences** switch, off by default. It is stored on the
device only: it is not synced and not written to the drive or the manifest.

- **On**: the app offers "New chord sequence" wherever it offers a new note (the New menu and the empty states), and
  chord sequences open as described above.
- **Off**: the app offers no way to create a chord sequence. Opening a note detected as a chord sequence asks whether
  to enable chord sequences. If the user agrees, the app turns the setting on and opens the note in the chord editor.
  Otherwise the note is not opened and the app stays where it was.

The setting never changes notes: detection, parsing, sync and every other note operation are the same whether it is on
or off, so turning it off hides nothing and loses nothing.

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
