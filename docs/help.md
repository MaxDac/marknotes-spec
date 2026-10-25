# Getting started and Markdown help

This document defines the shared content and acceptance criteria for the MarkNotes first-access guide and in-app
Markdown help. It describes the intended onboarding experience, not a claim that it has shipped. See
[the feature matrix](../features.md#privacy-diagnostics-and-app-information) for rollout status.

The Markdown coverage below describes existing renderers. Adding the guide does not add Markdown extensions or
change the [note format](../note-format.md), sync algorithm or fixture expectations.

## Getting started

### Your notes come first

MarkNotes keeps your notes locally so you can write without connecting a drive. A note is a plain-text `.md` file,
and folders organize your notes. The title is the file name without `.md`.

Edits save automatically on the device. **Autosave is not cloud sync.** Connecting a drive is optional; each connected
drive mirrors the same notes tree. Use **Sync now** to sync deliberately. Sync also runs at app-specific lifecycle
events; MarkNotes does not periodically sync while the app is closed.

Web also offers memory-only storage for shared browsers. In that mode, notes and settings are temporary and disappear
when the session ends. Connecting a drive and syncing is necessary if you want to keep those notes elsewhere.

### Find and organize your work

Use **New** or **New note** to create a note and the folder navigation to organize it. Import lets you turn supported
documents or text into notes; the available import actions depend on the platform.

Formatting controls insert Markdown into the note's source. Android offers **Edit** and **Preview**. Desktop and Web
also offer **Split**, with source and preview together. You can always type Markdown directly.

### Try a safe demonstration

The guide's editor demonstration is clearly labelled **Demo - changes are not saved**. It is not a note in your
workspace. You can try formatting and view controls without creating, editing or syncing a real note.

Note links, image insertion and drive setup are explained without importing real images, indexing real notes or
starting sign-in. Leaving the demo discards its text and returns you to your workspace.

Use this common starting body in the in-memory demo. The sandbox label and explanation remain visible outside the
editable body, so deleting the example text cannot hide the fact that it is a demo:

````markdown
# Welcome to MarkNotes

Try **bold**, *italic* and ~~strikethrough~~, then switch to Preview.

## A small plan

- Write an idea
- Put related notes in a folder

1. Edit the Markdown source
2. See the formatted preview

- [ ] Try a formatting control
- [x] Keep real notes unchanged

> This demonstration is temporary. Its changes are not saved.

Use `inline code`, or a fenced block:

```javascript
const greeting = "Hello";
```

| View | What it shows |
| --- | --- |
| Edit | Markdown source |
| Preview | Formatted text |
````

The shared body deliberately contains no real note links, image references, front matter or platform-only features.
Desktop and Web can explain Split in the surrounding guide without suggesting that Android has that view.

### Connect only the drives you need

| App | Available drive providers |
| --- | --- |
| Android Libre | Dropbox |
| Android Pro | Dropbox, OneDrive, Proton Drive |
| Desktop | Dropbox, OneDrive, Proton Drive |
| Web | Dropbox, OneDrive |

Actual setup also depends on the build's provider configuration and platform support. Existing provider notices and
sign-in restrictions still apply. The guide does not connect accounts, start sync or change privacy settings.

### Help is always available

Help contains **Getting started** to replay the guide and **Markdown guide** to read the syntax reference. Essential
help and demonstrations are bundled with the app and need no account or network connection once the app's assets are
available offline.

## What is Markdown?

Markdown is a way to format plain text using readable punctuation. For example, `**important**` becomes **important**
in the preview. The source remains text you can edit in another tool.

Markdown implementations differ. MarkNotes supports the common constructs below plus tables, strikethrough and
task lists; it does not promise every GitHub-flavored Markdown feature. A syntax-highlighting color in the editor
does not necessarily mean that syntax is rendered specially in Preview.

### Common syntax

| What you want | Source example | Notes |
| --- | --- | --- |
| A heading | `# Heading` | One to six `#` characters make H1-H6 |
| Bold | `**important**` | Formatting buttons insert the punctuation |
| Italic | `*emphasis*` | Can also be typed directly |
| Strikethrough | `~~old text~~` | Supported by all four apps |
| A bullet list | `- First item` | Indent to nest items |
| A numbered list | `1. First item` | Use separate lines for following items |
| A task list | `- [ ] To do` and `- [x] Done` | Change the source to check or uncheck a task |
| A quotation | `> Quoted text` | Start each quoted paragraph with `>` |
| Inline code | `` `code` `` | Displays code without interpreting its Markdown |
| A web link | `[Example](https://example.com)` | The example is a link, not an automatic network request |
| A horizontal rule | `---` | Put it between blank lines in the body |

Separate paragraphs with a blank line. A plain source newline does not necessarily force a visible line break.
The heading toolbar button cycles through H1, H2, H3 and plain text; type H4-H6 manually. Tables and horizontal
rules also use manual syntax rather than dedicated toolbar buttons.

Task-list checkboxes in Preview are read-only. They show the note's source state; they are not a second editing UI.

### Code blocks

Use three backticks on their own lines around a code block. An optional language name requests syntax highlighting:

````markdown
```javascript
const greeting = "Hello";
```
````

Indented code blocks are supported too. Highlighting depends on the app's bundled language grammars. A language that
is not recognized still displays as code, without a promise of colored syntax. Editor highlighting and preview
highlighting may use different language sets.

### Tables

Type a header, a separator row and the body:

```markdown
| Item | Status |
| --- | --- |
| Outline | Ready |
| Draft | In progress |
```

All four apps render this table. There is no dedicated table-insertion button.

## MarkNotes-specific features

### Links between notes

Type `@` at the start of a line or after whitespace to choose another note. MarkNotes inserts a link using that
note's stable ID, so it keeps working after a rename or move:

```markdown
[Project plan](marknotes:k7f3q2xa)
```

This is an explanation of the syntax, not a link to a bundled sample note. A link to an ID that does not exist is
broken. See [note links](../note-format.md#note-links) for the exact behavior.

### Images

Use the app's image-insertion actions to store an image as an attachment. These actions create a canonical
`attachments/<sha256>.<ext>` reference; arbitrary local file paths are not interchangeable with attachments.
Attachments resolve from the notes root, even for a note in a subfolder.

PNG, JPEG, GIF and WebP are supported. Non-GIF images are downscaled to a longest side of at most 2048 pixels; GIF
bytes are kept unchanged. The final stored image must be at most 10 MiB. See
[images and attachments](../note-format.md#images-attachments) for the complete rules.

Android and Web never load remote images. Desktop blocks them by default and offers a per-note, current-session
choice to load HTTPS images. Ordinary links and image loading are different capabilities.

### Metadata and chord sequences

MarkNotes stores a stable note ID in hidden front matter. It is preserved when saving and is not part of the body
you edit or preview. You do not need to write an ID when creating a note normally.

Chord sequences are a separate, optional editor feature, off by default. They are not a general Markdown extension.
See [chord sequences](../note-format.md#chord-sequences).

## Coverage and limitations

Libre and Pro use the same Android renderer through the shared Libre module.

| Capability | Android Libre and Pro | Desktop | Web |
| --- | --- | --- | --- |
| Common headings, emphasis, lists, quotes, links and code | Yes | Yes | Yes |
| Tables and strikethrough | Yes | Yes | Yes |
| Task-list rendering | Read-only checkboxes | Read-only checkboxes | Read-only checkboxes |
| Heading toolbar | H1-H3; type H4-H6 manually | H1-H3; type H4-H6 manually | H1-H3; type H4-H6 manually |
| Syntax-highlighted fenced code | Bundled Prism grammars | Bundled Highlight.js grammars | Bundled Highlight.js grammars |
| Edit and Preview | Yes | Yes, also Split | Yes, also Split |
| Bare-URL linkification | Not promised; use explicit Markdown links | Yes | No |
| Canonical local attachments | Yes | Yes | Yes |
| Remote images | Never loaded | HTTPS only after per-note session consent | Never loaded |
| Embedded raw HTML | Not a supported help capability | Disabled; source is escaped | Disabled; source is escaped |
| Math, Mermaid, footnotes and emoji-shortcode extensions | Not configured | Not configured | Not configured |

Do not mistake a code block labelled `latex` or `mermaid` for equation or diagram rendering. The guide must not add
plugins, enable HTML, relax image rules or change parser settings to make a documentation example work.

## Guide behavior and acceptance

### Local state

The automatic guide is offered once for both new and existing installations or browser profiles when no handled
flag exists. Explicit **Skip**, **Finish**, closing the guide, Escape or Android Back consume that automatic display.
Replay remains available and does not clear the flag. A later content revision alone does not trigger another tour.

Use each app's normal local settings store, including its encryption/storage mode. The flag is not synced.
Memory-only Web retains it only for the session. Normal sync-state cleanup must not reset it; Desktop's ordinary
local-data wipe retains it with app preferences. Full application-data removal, forgotten-key/database reset or
Web **Forget this device** removes it along with the relevant settings.

Do not treat lock, unexpected navigation, initialization failure, process death or a missing highlight target as
successful completion. If writing the flag fails, report the error rather than claiming it was saved.
Android configuration changes preserve the active step where the guide remains eligible.

### Readiness and interruptions

Start only after successful local setup/unlock and when the workspace is visible and stable. Defer for imports,
OAuth callbacks, dialogs, pending real-note saves and unfinished chord edits. Initial empty lists do not prove
initialization has finished. Neither successful cloud sync nor service-worker registration is required.

Coordinate in-flight operations and automatic reconnect scheduling without changing sync rules. On Web, use
cross-tab coordination to avoid two automatic tours in the same persistent browser profile.

### Non-saving means no note side effects

The demo owns separate in-memory state. Never send a fictional path through a production note-opening pipeline:
opening a note can assign IDs, create save callbacks, index notes or import attachments.

The demo must not write notes, assign real IDs, register pending saves or sync callbacks, import attachments, build
a real-note mention index, change the remembered active note or include text in diagnostics. Disable or sandbox
file paste/drop so events cannot reach the real workspace importer. Only the handled preference is an intended
onboarding storage mutation.

Preserve the real editor's text, selection, undo history, folder, note, view mode, scroll and focus. Leaving or
interrupting the guide destroys demo state and overlays. Lock must hide and clear demo content as well as protect
the real workspace.

### Accessibility and offline operation

Provide named Back, Next, Skip and Finish controls, readable progress, keyboard operation, focus confinement and
restoration, and screen-reader announcements. Respect reduced motion, large text, themes and touch targets.
Only highlight visible targets; reveal and restore a narrow-layout sidebar where needed. Missing or unavailable
controls must have an accurate explanatory alternative, not point to an invisible element.

Bundle scripts, styles and help content locally. Do not add analytics, CDN requests, sign-in requirements, CSP
relaxations or Electron privileges for onboarding. Core help is usable offline after the app assets are available.

### Verification

Each app tests automatic display, persistence, replay, interruptions and its storage/lock behavior. Verify the
documented examples using the actual preview renderer, not only editor highlighting or toolbar tests.

Compare notes, IDs, attachments, accounts, remembered active note and sync state before and after demo editing,
formatting, mentions, file paste/drop, exit, lock and concurrent sync. Also check real draft, selection, undo and
view restoration. Test keyboard/screen-reader behavior and small screens or windows, not just a default layout.
