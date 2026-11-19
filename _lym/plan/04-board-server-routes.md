# Card 4 — board-server-routes

**Create:**
- `internal/boardserve/doc.go`
- `plan:internal/boardserve#Server` -> `type Server struct`
- `plan:internal/boardserve#ServerConfig` -> `type ServerConfig struct`
- `plan:internal/boardserve#NewServer` -> `func NewServer(cfg ServerConfig) *Server`
- `plan:internal/boardserve#Server.ServeHTTP` -> `func (s *Server) ServeHTTP(w http.ResponseWriter, r *http.Request)`
- `plan:internal/boardserve#Server.BoardChanged` -> `func (s *Server) BoardChanged(revision string)`
- `plan:internal/boardserve#Server.beginDrain` -> `func (s *Server) beginDrain()`
- `plan:internal/boardserve#Server.lastAPIRequest` -> `func (s *Server) lastAPIRequest() time.Time`
- `plan:internal/boardserve#Server.applyWrite` -> `func (s *Server) applyWrite(req writeRequest) (boardengine.ViewEntry, error)`
- `plan:internal/boardserve#writeRequest` -> `type writeRequest struct`
- `plan:internal/boardserve#contentSecurityPolicy` -> `func contentSecurityPolicy(page []byte) string`
- `plan:internal/boardserve#routePage` -> `const routePage = "/"`
- `plan:internal/boardserve#routeBoard` -> `const routeBoard = "/api/board"`
- `plan:internal/boardserve#routeWrite` -> `const routeWrite = "/api/write"`
- `plan:internal/boardserve#routeFile` -> `const routeFile = "/file/"`
- `plan:internal/boardserve#maxBodyBytes` -> `const maxBodyBytes = 1 << 20`
- `internal/boardserve/testmain_test.go`
- `internal/boardserve/server_test.go`
- `plan:internal/boardserve#TestServer` -> `func TestServer(t *testing.T)`

**Uses:**
- `plan:internal/boardengine#Board.Expecting`
- `plan:internal/boardengine#Board.View`
- `plan:internal/boardengine#ViewerPage`
- `plan:internal/boardengine#BoardRevision`
- `plan:internal/boardengine#StaleEntryError`
- `internal/boardengine#Board.UpsertTaskReporting`
- `internal/boardengine#Board.SetStatusReleasingProject`
- `internal/boardengine#Board.PromoteTo`
- `internal/boardengine#IsRunStatus`

**Intent:**
Create `internal/boardserve`, the board server's HTTP handler (D9, D10, D13); its lifecycle follows in card 5.
`doc.go` documents the package: the routes, the checks and that it writes the board only through `boardengine`'s exported board methods.
`internal/boardserve/testmain_test.go` is untagged: a support-only file whose `TestMain` calls `gitkit.HermeticGitEnv()` and then `os.Exit(tmuxkit.Main(m))`, as every test package's does under `PATTERN-test-isolation`.

- `ServerConfig` holds the `*boardengine.Board`, the board path, the token, the bound port, the long-poll bound (production: 50 seconds) and a `Now` clock.
  `NewServer` computes the page's CSP once from `boardengine.ViewerPage()` and the current revision from `BoardRevision`.
- Checks, in order, for every request:
  1. `Host` must equal `127.0.0.1:<port>`, else 421 `{"ok":false,"error":...}`; this stops DNS rebinding.
  2. `/api` requests need `Authorization: Bearer <token>`, and the file route the `t` query parameter, compared in constant time; missing or wrong is 401.
     `GET /` needs no token, since the page holds no board data.
  3. A write needs `Content-Type: application/json` (415 otherwise), an `Origin`, when present, equal to `http://127.0.0.1:<port>` (403 otherwise), and a body within `maxBodyBytes` (413 otherwise, read through `http.MaxBytesReader`).
  Every response, refusals and 404s included, carries `Cache-Control: no-store` and `X-Content-Type-Options: nosniff`, and none carries a CORS header.
  These transport refusals name no way forward: the page never meets them with the token `open` handed it, and a page whose requests keep failing shows its `lym board open` line (card 3).
  Every `/api` request records its time through `Now`, which `lastAPIRequest` reads for the idle exit.
- Routes:
  - `GET /` (exactly `routePage`) serves `ViewerPage()` as `text/html` with `Content-Security-Policy`:
    `default-src 'none'; script-src <hash>; style-src <hash>; connect-src 'self'; base-uri 'none'; form-action 'none'; frame-ancestors 'none'`.
    `contentSecurityPolicy` hashes each inline `<script>` and `<style>` element's content as `'sha256-<base64>'`; it never emits `'unsafe-inline'`, so the page's `board-data.js` tag is blocked over `http`.
  - `GET /api/board` answers `{"ok":true,"revision":..,"board":<Board.View()>}`.
    With `after=<revision>` equal to the current revision it waits until `BoardChanged` sets another revision, the bound passes, the request ends or `beginDrain` runs;
    the bound and the drain answer `{"ok":true,"unchanged":true,"revision":..}`.
  - `GET /file/<name>` serves the file from the board path as `text/plain; charset=utf-8`, only when `name` equals a `design_file` or `decision_file` of the current `Board.View()`; any other name, one holding a separator included, is 404.
  - `POST /api/write` decodes a `writeRequest` (`op`, `slug`, `revision`, `fields`), refusing an unknown key, and runs `applyWrite`.
- `applyWrite` runs the one `boardengine` write of the op on `board.Expecting(slug, revision)`, whose write section checks the revision under the write lock (card 2):
  - `create`: `fields` may hold only `kind`, `title`, `labels`, `brief`, `body`, `project` and `depends_on`.
    A slug the current `Board.View()` already holds is refused as stale with that entry, as card 2's create refusal words it;
    otherwise `Expecting(slug, "")` and `UpsertTaskReporting`, whose write section refuses a slug another writer created in between.
  - `edit`: `fields` may hold only `title`, `labels`, `brief`, `body`, `project` and `depends_on`; `Expecting(slug, revision)` and `UpsertTaskReporting`, with no pre-check, so the write section alone decides a stale edit.
  - `promote`: `Expecting(slug, revision)` and `PromoteTo(slug, "", nil)`, the CLI's promote; the write section decides a stale promote, and `PromoteTo`'s own refusal answers a project.
    A task is refused before the write: `entry "<slug>" is a task already, and only a note is promoted: way forward: edit it, or promote a note`.
  - `release`, `draft` and `undraft` read the entry from the current `Board.View()` first, since their rule reads its kind and status;
    a missing entry, or a revision unequal to the request's, answers the stale refusal from that view without a write.
    Each refuses a non-task: `entry "<slug>" is a <kind>, and the page's <op> takes a task: way forward: set a note's or project's status with lym board set-status`.
    - `release`: a task whose status is a run status is refused, `entry "<slug>" is held by a run (status "<status>"), and a release from the page would let its parent's dobby start it while the run holds its pair: way forward: release it with lym board set-status`;
      else `SetStatusReleasingProject(slug, nil)`, the CLI's `set-status` null write.
    - `draft`: refused unless the task's status is empty, `draft takes a task with no status; entry "<slug>" has status "<status>": way forward: release it first, or leave it`;
      else `SetStatusReleasingProject(slug, &"draft")`, the write the CLI's `set-status` with `"draft"` makes.
    - `undraft`: refused unless the status is `draft`.
      A task with no status: `entry "<slug>" has no status, so it is released already and nothing was written`.
      A run status: `entry "<slug>" is held by a run (status "<status>"): way forward: clear it with lym board set-status`.
      Any other status: `undraft takes a draft task; entry "<slug>" has status "<status>": way forward: release it with the page's release`.
      Else `SetStatusReleasingProject(slug, nil)`.
  - an unknown op is refused naming the six ops.
  A field outside the op's list is refused naming the list.
  Success answers 200 with the entry from a fresh `Board.View()`; a `*StaleEntryError` answers 409 with its `Fresh` entry or null; any other error answers 422 with the `boardengine` refusal's own message, such as a run lock or a label rule.
- `BoardChanged` stores the revision and wakes every waiting long poll; `beginDrain` wakes every waiter at once with "unchanged" and makes later long polls answer at once.

Bound: the page reaches only these six writes, each through the same `boardengine` method the CLI verb calls, so the run lock, the label rules and the task-creation rule all run;
the server adds the revision check, the run-status release refusal and the task-only rule of release, draft and undraft, and drops no check.
The CLI's upsert also runs `refuseNoticeOnAbsentEntry` and `prepareRunSettings`, which act only on `notify_on_land`, `run_settings` and `profile`, fields no page op accepts.
Holding the token lets a same-user process write as the operator, which the token and loopback bound to the user's own processes (D9).

`TestServer`, tier0 in `internal/boardserve/server_test.go` (package `boardserve`), drives `ServeHTTP` in process with `httptest.NewRequest` over a temp board built from a `boardengine.Config` with `SkipGit`:
- refusals table, each changing nothing (board.json bytes compared): a wrong `Host`; an `/api` request without the token and with a wrong one; `GET /file/<name>` without `t` and with a wrong `t`; a write without `Content-Type: application/json`; a write with a foreign `Origin`; a body above the cap;
- every response carries `Cache-Control: no-store` and `X-Content-Type-Options: nosniff` and no `Access-Control-Allow-Origin`;
  `GET /` returns `ViewerPage()`'s bytes and a CSP with no `'unsafe-inline'` whose `script-src` and `style-src` hashes equal the base64 SHA-256 of the inline `<script>` and `<style>` content the test extracts from `ViewerPage()`;
- `GET /api/board` returns, render time aside, what `Board.View()` builds over the same board;
- ops table: create, edit, release, promote, draft and undraft through `POST /api/write` each leave `board.json`, dates aside, as the boardengine call the matching CLI verb's RunE makes leaves a twin board, and answer the fresh entry;
- an edit carrying the `revision` of the entry a successful write answered is saved;
- refused and changing nothing: draft of a task with a status; undraft of a task with no status, of one in a run status and of one with another status, each with its message; draft and undraft of a note; release of a task in a run status (naming `lym board set-status`); release of a note; promote of a task; an unknown op; an unknown field;
- a create of an existing slug answers 409 with the existing entry;
- an edit of a running task's title is refused with the run lock's message, as the CLI refuses it;
- an edit carrying an older revision answers 409 with the fresh entry, refused inside the write section; a release carrying an older revision answers 409 from the view, and one of a removed entry 409 with a null entry;
- long poll: with the current revision and a short bound it answers "unchanged"; a `BoardChanged` during the wait returns the new projection; `beginDrain` answers a waiter "unchanged" at once;
- `GET /file/<name>` serves a listed design file and answers 404 for an unlisted name and for `a/b.md`;
- the page holds each route constant's path.

**Commit:** 4: boardserve: serve the board page, its data and its writes
