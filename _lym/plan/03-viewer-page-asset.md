# Card 3 — viewer-page-asset

**Create:**
- `internal/boardengine/board.html`
- `internal/boardengine#BoardPageFile`
- `internal/boardengine#viewerPage`
- `internal/boardengine#ViewerPage`
- `internal/boardengine_test#TestViewerPage`

**Edit:**
- `internal/boardengine#renderFiles`
- `internal/boardengine#FabricReplay.Claims`
- `internal/boardengine#TestFabricReplay_Ledger`
- `internal/boardengine_test#TestRenderToDiskManifestCleanup`
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#BoardDataFile`
- `internal/boardengine#RenderToDisk`

**Intent:**
Ship the viewer page (D2, D4, the page half of D10, D11's retry) as an embedded asset and a fixed render output, and claim both new files in the Bolt replay (D3).

- `internal/boardengine/board.html` is one self-contained HTML file: one inline `<style>` element, one inline `<script>` element, and `<script src="board-data.js">` before it.
  It carries no inline event handler and no `style` attribute; the script sets styles through the CSSOM and wires events with `addEventListener`.
  It holds no `http://` or `https://` resource URL and loads nothing beyond its own origin; it carries no CSP `<meta>`.
- A new `viewer.go` declares `viewerPage` with `//go:embed board.html`, `ViewerPage` returning a copy, and `BoardPageFile`.
  `renderFiles` adds `BoardPageFile` with `viewerPage` verbatim, so a render rewrites identical bytes and a changed board leaves the file unchanged.
- Modes, picked by `location.protocol` before `window.BOARD` is read:
  - at `file:`: the page reads `window.BOARD` only; read-only, with no edit control and no live refresh.
    When `window.BOARD` is undefined, it shows one line naming `lym board rerender`.
    Design and decision links are `./<name>` relative paths.
  - over `http:`: the page never reads `window.BOARD`.
    It reads the token from the URL fragment, keeps it in `sessionStorage` for the tab, and drops the fragment with `history.replaceState`.
    It calls `GET /api/board`, then long-polls `GET /api/board?after=<revision>`, each with `Authorization: Bearer <token>`; the server answers within a minute, so the page asks at least once a minute.
    Answers: `{"ok":true,"revision":...,"board":<BoardView>}` or `{"ok":true,"unchanged":true,"revision":...}`.
    Writes go to `POST /api/write` with `Content-Type: application/json` and the body `{"op":..,"slug":..,"revision":..,"fields":{..}}`;
    the ops are `create`, `edit`, `release`, `promote`, `draft` and `undraft`; `create` sends no revision.
    A success answers `{"ok":true,"entry":<ViewEntry>}`; a stale refusal answers `{"ok":false,"error":..,"entry":<ViewEntry or null>}`; any other refusal `{"ok":false,"error":..}`.
    Design and decision links use `/file/<name>?t=<token>`.
    A failed request retries with backoff from one second doubling to eight, for a bounded minute, and only then shows one line naming `lym board open`.
- Viewer (D4): kind, type-label and area-label toggle groups, OR inside a group and AND across groups, an empty group filtering nothing;
  case-insensitive free-text search over handle, slug, title, brief and body;
  grouping by project as the README nests them (a sub-project under its `parent`), each project's entries grouped by `section` in the order of `sections`, with each group's count after filtering and an empty group hidden;
  an entry whose `project` names no listed project is shown under the root's group;
  a row per entry with handle, title, status and labels that expands to the brief, a facts line (slug, kind, owner, recipe, ages against the browser's clock) and the body;
  the header shows `rendered` and `prime_owner`; light and dark follow `prefers-color-scheme`.
- The body renders a Markdown subset: headings, paragraphs, bullet and numbered lists, fenced code, code spans, bold, italic and links.
  Every body is HTML-escaped first, double quotes included.
  A link target has leading and trailing C0 controls and spaces trimmed and every tab, carriage return and line feed removed;
  it is kept only with the scheme `http` or `https`, compared case-insensitively, or with no scheme and no opening pair of characters that are each `/` or `\`;
  every other target, `javascript:`, `data:`, `//host` and `/\host` among them, renders as plain text.
  Every other projection value and every refusal message is inserted with `textContent`, and a file name reaches a link only as its `href` set through the DOM.
- Page writes (D10): a create form (slug, kind, title, labels, brief, body, project, and depends_on for a task),
  an edit form on an expanded entry (title, labels, brief, body, project, and depends_on for a task),
  and only the actions the entry's kind and status allow: release for a task whose status is neither empty nor a run status (`<state> · <producer>`), promote for a note, draft for a task with no status, undraft for a draft task.
  On a stale refusal the page shows the fresh entry beside the operator's unsent edit, which stays in its form; each refusal's message is shown as text.
  The revision a form sends is the one the page holds for the entry: a stale refusal replaces it with the fresh entry's `revision`, so a resend after the operator compared goes through, and a success replaces it with the answered entry's `revision`, so the next edit carries it.
  A stale refusal with a null entry removes the entry from the view and keeps the unsent edit's text in its form.
- `FabricReplay.Claims` claims `board-data.js` and `board.html` at the Bolt root, through `BoardDataFile` and `BoardPageFile`.
- Every board write already runs `RenderToDisk` and then the sync, which stages the whole Bolt tree, as `TestSync_Scenario`'s dirty-board step pins, so both files land in the same `board sync` commit as `README.md` with no change to the sync.
- `doc.go`'s `# Replay` list of claimed paths names the two files, and the `# README` section names `board.html` as a fixed render output.
  That section's sentence on what every write renders also names `board-data.js` and `board.html`.
- `render.go`'s file header also names `board.html` among Render's outputs,
  and the comment over `replay.go`'s file-name constants names `board-data.js` and `board.html` among the files a replay recognises at the Bolt root.

The page's in-browser behaviour is a review check, since the suite has no JavaScript runtime.

`TestViewerPage`, a new tier0 test in `internal/boardengine_test`, asserts:
`Render`'s `board.html` equals `ViewerPage()` byte for byte, and a second render of a changed board gives the same bytes;
the page references `board-data.js` through a `<script src>` tag and holds no `http://` or `https://`.
`TestRenderToDiskManifestCleanup`'s seeded-manifest row, which card 1 made assert `board-data.js`, asserts `board.html` is written and listed in the manifest too.
`TestFabricReplay_Ledger`'s `Claims` table gains rows: `board-data.js` and `board.html` claimed, `sub/board.html` not.

**ImpactSummary:** boardengine only: every render writes one more fixed file, and the replay claims two more paths; no signature changes.

**Commit:** 3: board: embed the viewer page and claim its files in the replay
