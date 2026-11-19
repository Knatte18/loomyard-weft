# Decision record — board-viewer-build

## Goal

Give the operator an interactive view of the board, with filters, opened from VS Code by one key instead of a typed lym command.
The view also edits the board: it creates and edits tasks and notes, releases tasks, promotes notes, and drafts and undrafts tasks.
It refreshes live when anything else writes the board, its writes go through `internal/boardengine` like every other writer,
and a write carries the revision of the entry it edited, so a stale write is refused and the page shows the fresh entry.

## Scope

In:

- `board-data.js` and `board.html` as render outputs of the board, written, committed and replayed with the README.
- The viewer page: filters, free-text search, grouping, expandable entries, light and dark, edit forms, live refresh.
- A per-entry revision checked inside the board's write section, and the stale-write refusal carrying the fresh entry.
- A per-hub loopback board server, `lym board serve`, that serves the page, the board, its changes and the page's writes.
- A `lym board open` verb that makes sure the hub's server runs and opens the page in the default browser.
- VS Code tasks `board open` and `beam start`, and two key bindings:
  `lym ide seed` and `lym ide spawn` write `board open` and the keys,
  and the hub workspace regeneration writes `beam start` through the ide module's workspace-content hook.

Out:

- Window titles, the folderOpen chain and `launch.json` from the entry's `Was ide-prime-setup` part.
- A Live Preview panel inside VS Code: the task opens the default browser only.
- macOS: lym's platform files cover Linux and Windows only, as `internal/vscode/launch_linux.go` and `internal/vscode/launch_windows.go` show.
- Page actions beyond the entry's list: no remove, rename, merge, slug change, kind change, project create or edit, `run_settings` or profile edit, decision-record upload or prune.
- Serving the board to another machine: the server listens on loopback only.

## Decisions

### D1 — `board-data.js` is a render output built from the README's own model

`boardengine.Render` returns `board-data.js` beside the README in its file map.
Its content is `window.BOARD = <json>;` and a newline.
The JSON is a view projection computed by the same code that builds the README, so the viewer recomputes nothing:

- top level: the render time, the prime owner, the configured type labels in order, the configured area labels,
  the README's section names in the README's order, a project list and an entry list;
- per project, the root `P0` included: handle, slug, title, brief, labels, the parent project's handle, and its design file name;
- per task or note: handle, slug, title, kind, type labels and area labels split by the board's vocabulary, status, its project's handle,
  the README section it sits in (Running, Ready, a layer, Independent, Draft, a Notes type, Done),
  the handles it waits on, brief, body, owner, recipe, created and revised timestamps,
  the design and decision file names when they exist, and the entry's revision (D8).

Each entry's revision is computed over the stored entry, before the render's copy that clears `Discussed` for an entry whose decision file is missing;
the projection's decision-file name follows that copy, so no missing file is listed.

Timestamps are absolute; the viewer computes ages against the browser's clock.
The JSON is encoded with Go's default HTML escaping, so a `<`, `>` or `&` in any field is written as a JSON unicode escape, a backslash and `u` followed by the hex code `003c`, `003e` or `0026`,
and a body holding `</script>` cannot end the script.
The board server (D9) serves the projection built by the same function, over a store loaded as D8 states.

Rationale: a projection keeps layering, handles, sections and project nesting in Go, where the README already computes them,
so the viewer and the README never disagree on where an entry sits, and the file and the server never disagree with each other.

### D2 — `board.html` is an embedded asset written as a fixed render output

The viewer is one self-contained HTML file with inline CSS and JavaScript, embedded in the binary with `go:embed` in `internal/boardengine`.
`boardengine.Render` returns it verbatim as `board.html`.
The board server serves the same embedded bytes and builds the projection itself, so it never reads the two committed files and a stale one never reaches it.
Rewriting identical bytes leaves no git change, so with one lym version on a hub the file changes in the Bolt only when a new binary ships a changed viewer.
The page picks its mode by `location.protocol` before it reads `window.BOARD`.
Opened at `file://`, the page loads its data only through `<script src="board-data.js">`, since a browser blocks `fetch` from a page opened at `file://`:
the Fetch Standard leaves a `file:` fetch to the browser and advises a network error when in doubt,
Chromium refuses the `file` scheme in `fetch`, and Firefox gives each file its own origin.
It is then read-only, with no edit control and no live refresh.
Served over `http`, it reads and writes through the server's API (D10) and never reads `window.BOARD`;
the CSP header (D9) blocks its `board-data.js` script tag there.
It loads nothing from the network beyond its own origin.

Both file names are named constants in `internal/boardengine`.

Rationale: a render output needs no separate install step, survives a fresh Bolt clone, and is removed and committed through the existing render manifest and sync path;
one asset for both modes keeps one page to review.

### D3 — the new files ride the existing write, commit and replay paths

- `boardengine.RenderToDisk` writes both files, and its render manifest lists them, so stale-file removal covers them.
- Every board write runs `RenderToDisk` and then the sync, which stages the whole Bolt tree, so both files land in the same `board sync` commit as `README.md`.
  The owner-set write path commits every name `Render` returns, so it carries them too.
- `boardengine.FabricReplay.Claims` claims both names, as it claims `README.md`, so a Bolt replay re-renders them;
  a commit touching an unclaimed path is replayed as a plain cherry-pick.

Bound: these paths carry the README's own exposure to mixed lym versions on one hub.
A machine on an older lym leaves `board-data.js` stale until a newer one writes;
two binaries with different viewers rewrite `board.html` on each write;
and a replayed commit that changed only rendered files is replayed as empty, so its new `board.html` waits for the next board write.
Each of these heals on the next write by an up-to-date binary.
An older binary claims neither file, and every commit of a newer binary touches `board-data.js`,
so the older binary replays every such commit as a plain cherry-pick, without the by-entry merge of `board.json`;
its concurrent board writes then hit the diverged-Bolt refusal below until that machine is upgraded.
A downgrade removes both files and commits the removal,
and since the older binary claims neither file, that commit replays as a plain cherry-pick rather than through the by-entry merge,
and it conflicts whenever a newer binary has written `board-data.js` since.
A conflicted replay restores the starting tip, and the board write is skipped as a diverged Bolt with the existing records-conflict way forward,
which names the conflicting paths;
the case recurs on each board write by the older binary until that machine is upgraded.
Since `board-data.js` carries the render time, every board write's sync commit carries it.

### D4 — the viewer page

- Filters: kind (task, note), type labels and area labels, each a set of toggles.
  Toggles in one group are OR'd, groups are AND'd, and an empty group filters nothing.
- Free-text search, case-insensitive, over handle, slug, title, brief and body.
- Grouping mirrors the README: each project a group, a sub-project nested under its parent,
  and a project's own tasks and notes grouped by section in the order D1's section list gives.
  Each group shows its count after filtering, and an empty group is hidden.
- Each entry is a row with handle, title, status and labels; clicking it expands the brief, the facts line and the body.
  A discussed entry links to its `decision-<slug>.md`, and an entry to its design file when the data names one.
  At `file://` the link is a relative path opening with `./`; over `http` it is the server's file route (D10).
- The body renders a Markdown subset: headings, paragraphs, bullet and numbered lists, fenced code, code spans, bold, italic and links.
  Every body is HTML-escaped before the subset applies;
  the escape covers double quotes, so an escaped target cannot leave its `href` attribute.
  A link's target first has leading whitespace and control characters trimmed,
  and every tab, carriage return and line feed removed, as the WHATWG URL Standard's basic URL parser does:
  it strips leading and trailing C0 controls and spaces from its input, and removes every ASCII tab or newline from it.
  It is kept only when it then has the scheme `http` or `https`, compared case-insensitively,
  or has no scheme and does not open with two characters that are each a slash or a backslash.
  Every other target, `javascript:`, `data:`, `//host` and `/\host` among them, renders as plain text.
- Every other value the page renders from the projection is inserted as text, never as markup,
  among them the header's render time and owner, each project's handle, title, brief and labels,
  each entry's handle, slug, title, brief, status and labels, the facts line, section names,
  the design and decision file names, and every refusal message the server returns.
  A design or decision file name reaches a link only as its `href` attribute set through the DOM, never through markup.
- Light and dark follow `prefers-color-scheme`.
- A header shows the render time and the prime owner.
- At `file://`, when `window.BOARD` is undefined, because `board-data.js` is missing or did not parse, the page shows one line naming `lym board rerender`.

### D5 — `lym board open` makes sure the server runs and opens the page

A new operator verb `lym board open` opens the hub's board page in the default browser.

- The board subtree's `PersistentPreRunE` (`internal/boardcli/cli.go`) resolves the `open` verb's cwd with `lymcwd.Resolve`, as it does for the other board verbs;
  the VS Code task's cwd is the workspace folder, which is the anchored directory.
  The verb takes no argument and refuses `--board-path`, naming a run from a worktree of the hub as the way forward, since the server needs the hub's config.
- It reads whether a server runs by the server's lock (D11), tried without blocking and released at once, as `dobbyengine.Live` reads a dobby's.
  A free lock means no server, whatever a leftover state file says.
  A held lock means a server that is starting or serving:
  `open` waits, bounded as the start path does, re-reading the state on each try, until the state answers a request carrying its token.
  `open` takes a state only when its recorded pid is a running process.
  A server removes any leftover state file right after it takes the lock and writes its own only once its port is bound,
  and an exiting server stops taking requests before it releases the lock (D11), so the state that answers is the live server's.
- When no server runs, it starts one (D11) and waits, bounded, until the lock is held and the state the new server wrote answers a request carrying its token.
  It starts the server with the prime's anchored directory as its working directory,
  taken through `lymcwd.NamedWorktree` for the prime `fabricengine.PrimeName` names, whatever worktree `open` ran from.
  When the prime does not resolve, `open` returns the error `lymcwd.NamedWorktree` gives and starts nothing.
- On either path, a server that does not answer in the bound is refused with the server log's path named.
  The log opens with the server's pid (D11), so the operator can end a hung server, which frees its lock for the next `open`.
- It writes a launcher file (D9) and hands the opener the launcher's path.
- The opener is a per-platform build file: `xdg-open <path>` on Linux, `rundll32 url.dll,FileProtocolHandler <path>` on Windows.
  The spawn is logged, detached with `proc.Detach` and reaped with `proc.Reap` as the server's is, and waited on within the bound below.
- A failed opener, such as a Linux machine without `xdg-open`, returns its error with the opener named.
- On success it prints, in its JSON envelope, the page's address without the token, the launcher's path, and whether it started the server.
- In a run agent's session it refuses and starts nothing (D12).

Bound: `xdg-open` with no desktop detected runs a browser in the foreground,
as the generic mode of xdg-utils 1.2.1's `xdg-open` (`open_generic`) runs the handler it finds in the foreground and exits only when it exits:
the MIME type's desktop entry, `run-mailcap` or `mimeopen`, and failing those the first browser of `$BROWSER` or its built-in list.
The wait on the opener can so last until that browser exits.
For a caller with a strand name, an orch session among them, that wait is capped at the cap `shuttleengines`' keep-warm resolver gives the caller's role;
at the cap the verb leaves a still-running opener to finish on its own and returns success.
The operator's call, with no strand name, waits until the opener exits.
While it waits, the `board open` task stays active, and a second key press is answered by VS Code's running-task prompt.
The launcher reaches the page only when the platform's handler for an HTML file is a browser that runs its script and can read the launcher's path;
a browser confined from that path, or a non-browser handler, shows nothing while the verb reports success,
and the operator can open the printed launcher path in a browser by hand.

Rationale: lym resolves the hub path, so nothing is linked into a worktree;
the same verb serves a terminal and the VS Code task, and a second press reuses the running server.

### D6 — VS Code tasks and key bindings

- `board open`, running the lym binary with `board open`, goes into every worktree's `.vscode/tasks.json` beside `reed attach`,
  written by `vscode.WriteConfig` from `lym ide seed` and `lym ide spawn`.
  Its presentation sets `"reveal": "always"` and `"focus": false`:
  the terminal shows the verb's output, so D5's refusals and a failed opener's error are seen, and it does not take focus.
  The terminal panel therefore comes forward on every press, a successful one included.
- `beam start` goes into the workspace files' `tasks` key, never into a folder's `tasks.json`, so a multi-root window holds exactly one.
  VS Code's Multi-root Workspaces documentation, Tasks section, states that workspace-scoped tasks live in the `tasks` section of the workspace file
  and that only `shell` and `process` tasks may be defined there, so `beam start` is a `process` task.
  Per workspace file:
  the hub's `<RepoName>.code-workspace` runs `lym beam start root` and each beam's `<RepoName>-<beam>.code-workspace` runs `lym beam start <beam>`,
  each with the prime's or that beam's anchor as its working directory.
  It is presented like `reed attach`: revealed, focused, in a new panel.
- Fabric learns nothing of tasks, lym verbs or the lym binary.
  `fabricengine` offers a general workspace-content seam, installed process-wide as `fabricengine.InstallPairFolderNamer` is.
  The installed hook is handed the hub path, the beam slug a workspace file belongs to (empty for the hub's own file) and that worktree's anchor path,
  and returns extra top-level members as raw JSON keyed by name.
  `fabricengine.RegenerateHubWorkspace` reads only the members' names, to refuse one named as the beam file's generated mark, which fabric owns,
  and passes the members on to `internal/vscode`, whose workspace builder behind `vscode.BuildWorkspace` splices them verbatim after `settings`.
  Fabric so names no workspace key but its own mark, and every caller of the regeneration writes the members; with no hook installed a file carries no extra member.
  The builder refuses a member named as a key it writes itself (`folders` or `settings`).
  The regeneration builds every workspace file before it writes any, so either refusal is a programmer error it returns having written nothing.
  `cmd/lym/main.go` installs the ide module's hook beside the pair folder namer;
  the hook builds the `tasks` member through `internal/vscode`.
- Both commands run the absolute path `selfexec.LymBinary` returns in the writing process, through the ide module's `LymBinary` seam (`internal/ideengine/spawn.go`),
  and fall back to bare `lym` when it errors, as `vscode.WriteConfig` does for `reed attach`.
- The `// lym:begin` … `// lym:end` block in the user `keybindings.json`, written by the existing writer in `internal/vscode/keybindings.go`, gains two entries:
  `alt+shift+0` runs the task `beam start` and `alt+shift+9` runs `board open`, through `workbench.action.tasks.runTask`.
  Every seed or spawn rewrites them with the block.
- Task labels follow the existing `reed attach` style: the verb's words after `lym`.

`workbench.action.tasks.runTask` with a label runs the first task of that label it finds, searching the folders in order, then the workspace file, then the user's tasks, with no picker;
when no task carries the label it opens the task picker filtered by the label.
The pin is `_runTaskCommand` in `src/vs/workbench/contrib/tasks/browser/abstractTaskService.ts`, read on VS Code's `main` branch on 2026-11-19.

Bound: the key bindings are user-global, so a key can name a task its window lacks, and VS Code then opens its task picker filtered by that label.
That happens for `alt+shift+0` in a task pair's bare-folder window, in a prime or beam window opened as a bare folder because the prime did not resolve,
and in a window seeded before the upgrade until it is seeded again.
It happens for `alt+shift+9` in any window whose worktree has a git-tracked `.vscode/tasks.json`, which seeding skips today,
so that window lacks `board open` as it lacks `reed attach`,
and in a window seeded before the upgrade until it is seeded again.
A binary older than this change that regenerates the workspace files, or seeds or spawns,
rewrites them without the `beam start` tasks, the `board open` task and the two keys.
The next regeneration by an up-to-date binary restores the `beam start` tasks,
and its next seed or spawn restores `board open` and the keys, as D3 states for the board files.
In a multi-root window several folders carry `board open`, and VS Code runs the first folder's copy;
every copy runs the same verb on the same hub and starts a server in the prime's anchored directory (D5), so whichever copy VS Code runs, the result is the same.

Rationale: tasks are the mechanism the `reed attach` binding already uses;
a workspace-level `beam start` with an explicit slug starts the beam the window belongs to.

### D8 — an entry's revision is the hash of its stored record, checked inside the write section

- An entry's revision is the hex SHA-256 of the entry's JSON encoding, computed by one `boardengine` function.
  It covers every stored field, status included.
  Nothing new is stored: the revision is derived on read, so `board.json`'s shape does not change.
- The function always hashes the entry as the write section holds it after its load migrations (`migrateDraftStatus`, `migrateDiscussedStatus`),
  and the server's projection loads its store through that same load-and-migrate step, unsaved,
  so the revision a page holds and the revision the check computes are taken over the same bytes, even on a board an older binary left unmigrated.
  A write's answer computes the revision by the same function over the saved entry as a fresh load and migration reads it.
- A `boardengine` write the server makes carries the expected revision of each entry it edits.
  The write section compares it with the entry as loaded and migrated under the board's write lock, before the mutation runs.
  A mismatch, or an entry that no longer exists, refuses the write with a stale-entry error that carries the fresh entry's projection, or none when it was removed,
  and nothing is saved, rendered or synced.
- A create carries no revision and is refused when the slug already exists, with the existing entry's projection in the refusal.
  A slug refused for any other reason, such as one a raddle store holds, is answered with the board's own refusal message and no entry, as D10's other refusals are.
- The CLI verbs gain no revision flag and behave as today.

Bound: the check covers only the entries the write names.
A write the board makes to a sibling as a side effect, such as a project's release when its last member is released, is not checked against a revision,
exactly as the CLI's same write is not.

Rationale: a derived hash needs no schema change, no migration and no tolerant-record handling,
and checking under the write lock makes the compare and the write one atomic step against every other writer.

### D9 — the board server listens on loopback and trusts only its token

- `lym board serve` is a hub-level process, one per hub on a machine, listening on `127.0.0.1` on a port the OS picks.
- At start it draws a random token of at least 128 bits and writes its state (D11): pid, port and token, in a file only the user can read,
  written with mode 0600 and, on Windows, with an owner-only DACL, as `internal/githubclient/cache_windows.go` sets on the token cache.
  The DACL is needed because Windows ignores Go's permission bits, as `internal/githubclient/doc.go` records.
- Every request must carry a `Host` header equal to `127.0.0.1:<port>`, else it is refused; this stops DNS rebinding.
  Every address lym builds for the page, the launcher's and the printed one, uses `127.0.0.1`, never `localhost`.
- Every `/api` request must carry the token in an `Authorization: Bearer` header, and the file route (D10) in a `t` query parameter; a missing or wrong token is refused.
  The page itself, `/`, is served without a token, since it holds no board data.
- A write request must also carry `Content-Type: application/json`, and an `Origin` header, when present, must equal the server's own origin;
  the server sends no CORS header, so a cross-origin page can neither read a response nor send a write without a preflight it never answers.
- Every response carries `Cache-Control: no-store` and `X-Content-Type-Options: nosniff`;
  the page's response carries a `Content-Security-Policy` header that allows only its own inline script and style by their hashes, computed from the embedded asset, and connects only to its own origin.
  The policy is a response header only, never a `<meta>` in `board.html`, so it does not block `board-data.js` at `file://`.
  Under CSP Level 3, a hash source matches only an inline `<script>` or `<style>` element, and an inline event handler or a style attribute needs `'unsafe-hashes'`;
  the page so carries no inline event handler and no style attribute, and sets styles through the CSSOM.
- A request body above a fixed byte cap is refused.
- `lym board open` passes the token to the browser through a launcher file it writes under the server's state directory with mode 0600 and, on Windows, the state file's owner-only DACL:
  a small HTML file that replaces its location with the page's address holding the token in the URL fragment.
  The opener's command line so carries a file path, never the token.
  The page reads the token from the fragment and keeps it for the tab.

Bound: anything that runs as the same user on the machine can read the state file and write the board through the server as the operator.
That includes a run's agent: holding the token is enough, where today it would have to clear its strand-name variable to pass the board's task-creation guard.
The guard was never a boundary against a same-user process, and the token keeps out other users and every web page.
The server reaches only `boardengine`'s writes (D10), so it lets through nothing the CLI's operator writes do not.

Rationale: a loopback port with a per-start token, a Host check and no CORS is the standard defence for a local web tool.

### D10 — the server's routes and the page's writes

- `GET /` serves the embedded page (D2).
- `GET /api/board` returns the projection (D1) and the board's revision, the hex SHA-256 of `board.json`'s bytes.
  With `after=<board revision>` it waits, bounded under a minute, until the board's revision differs, then returns the fresh projection;
  at the bound it answers "unchanged" and the page asks again.
- `GET /file/<name>` serves a design or decision file from the Bolt as `text/plain`, only for a name the current projection lists; any other name is not found.
- `POST /api/write` takes one operation with its slug and the expected revision (D8), and runs it through the matching `boardengine` write, render and sync included:
  - create a task or a note: slug, kind, title, labels, brief, body, project and, for a task, the slugs it depends on;
  - edit a task or a note: title, labels, brief, body, project and, for a task, the slugs it depends on;
  - release a task whose status is not a run status: its status set to none, through the same write `lym board set-status` with `null` makes.
    A task whose status is a run status (`<state> · <producer>`, as `boardengine.IsRunStatus` reads it) is refused, naming `lym board set-status` as the way forward;
  - promote a note: the same write `lym board promote` makes;
  - draft a task whose status is none: status `draft`; undraft a task whose status is `draft`: status none.
    Draft and undraft refuse a task in any other status.
  Each answers with the entry's fresh projection on success,
  a conflict with the fresh entry on a stale revision (D8),
  and the `boardengine` refusal's own message for any other refusal, such as a run lock or a label rule.
- The page shows a create form, an edit form on an expanded entry, and the release, promote, draft and undraft actions an entry's kind and status allow.
  On a stale refusal it shows the fresh entry beside the operator's unsent edit, which stays in its form.
  Each refusal's message is shown as text.

Bound: the page reaches only these operations, and each runs the same `boardengine` method and checks the CLI's verb runs,
the run lock, the label rules and the task-creation rule among them;
the server adds the revision check and the refusal to release a task in a run status, and drops none.
Release refuses a task in a run status, since a none status would make it startable for its parent's dobby while its run still holds its pair;
the CLI's `set-status` stays the way to release one.
Promote acts as the CLI's does, so promoting a discussed Sketch note releases it into a raddle run in the same write.
The live view follows `board.json` alone: a change that leaves it untouched, such as a decision file rewritten for an entry already discussed, reaches an open page at the next change to `board.json` or a reload.

Rationale: one write route per board operation keeps the server a thin caller of `boardengine`;
the file route lets an `http` page open the files the `file://` page links by relative path.

### D11 — the server's lifecycle

- Its state directory is a new directory under the hub's board-level `.lym`, named by a `lymdirs` constant, holding the state file, the lock, the launcher and the server log.
  That `.lym` is never staged by the board sync, as creel's journal and lock beside it are not, so the token never reaches a commit.
- `lym board serve` refuses `--board-path` as `open` does, and refuses in a run agent's session (D12).
  It takes the directory's lock, waiting a few seconds so that `open`'s instant liveness read never turns it away;
  a server that still finds the lock held exits at once with success, as creel's daemon does, and the opener's wait then finds the winner.
  Right after it takes the lock it logs its pid and removes any leftover state file.
  It resolves its worktree as the other writing board verbs do, applies the board's skip variables as they do,
  builds its `boardengine` board once,
  registers itself with `runproc.SelfRecord` under `runproc.HubOwner` in a new `runproc` role for the board server, kept out of `subjectPassSpared`,
  binds its port, then writes its state and serves.
- `lym board open` starts it as creel's client starts creel's daemon: through `selfexec.LymBinary`, detached with `proc.Detach`, logged, reaped with `proc.Reap`,
  with its stderr in the server log, and with the prime's anchored directory as its working directory (D5).
- One watch loop drives change detection, binary replacement and idle exit.
  It wakes on an `internal/fswatch` event on `board.json` in the Bolt, or on a timer from one second doubling to eight seconds while the board is unchanged.
  When the watch cannot be set up, the timer alone reads `board.json`.
  A change recomputes the board's revision and wakes every waiting `GET /api/board`.
- A Bolt replay or a sync pull that rewrites `board.json` is a change like a local write, so the page shows other machines' writes once they reach this Bolt.
- The server exits when its binary has been replaced, through the watcher `selfexec.New` returns as creel's daemon does,
  and when no `/api` request has arrived for 30 minutes.
  An open page asks at least once a minute, so it keeps the server alive.
  On either exit it stops taking new requests, then releases its lock, leaving its state file for the next server to remove,
  and wakes every waiting `GET /api/board` at once with "unchanged";
  it then waits for the requests in flight, so a write inside the board's write section completes and spawns its sync.
  An `open` during that drain finds the lock free and starts a fresh server.
  A hub takeover by another machine ends it with the hub's other processes, through `runproc`, since its role is outside `subjectPassSpared`.
- A page whose request fails retries with backoff for a bounded time and only then shows one line naming `lym board open`.

Bound: the server's `boardengine` config and environment are read once at its start,
so a change to the hub's `board.yaml` takes effect at the next server start,
and a server started with a board skip variable set skips for every page write until it exits.
`open` starts every server in the prime's anchored directory, whatever worktree `open` ran from, so removing a task pair's worktree never takes away the layout the server's config reads.
The server's board config is so the prime's, and a config layer a beam's anchor adds does not apply to page writes.
A page's live view reaches only writes that land in this machine's Bolt.

Rationale: the creel daemon is lym's proven single-instance, on-demand pattern;
the idle exit and the binary check bound the server's life without an operator stop verb.

### D12 — the viewer is an operator tool

`lym board open` and `lym board serve` refuse in a run agent's session, the caller `boardengine`'s run-agent rule (`internal/boardengine/taskguard.go`) refuses a task creation from.
An orch session may run both.
The server passes the board an empty strand name, so its writes are the operator's.

Bound: the refusal stops a run's agent from starting a server or opening the page, and no more:
an agent that reads a running server's state file writes as the operator, as D9's bound states,
and a session that clears its strand-name variable passes the refusal, as it passes `lym board upsert`.

Rationale: a server an agent started would carry the agent's session into the operator's page.

### D13 — module boundaries

- `internal/boardengine` holds the projection, the embedded page, the revision function and the revision check.
- A new package `internal/boardserve` (new) holds the HTTP server, its routes, token and Host checks, the state file, the launcher and the watch loop.
  It calls only `boardengine`'s exported board methods to write.
- `internal/boardcli` holds the `open` and `serve` verbs, the opener seam and the per-platform opener files.
- `internal/runproc` gains the board server's role (D11).

Rationale: the server is a lifecycle and transport concern that `boardengine` should not carry,
and keeping it out of `boardcli` lets the verbs stay thin wiring.

### Added after Discussion (arbiter, 2026-11-17): Per-body README folds are moot

Decision: The phone part's collapsible blocks per body are not built in this task: README.md and every project-<slug>.md keep entry bodies out, in design-<slug>.md linked from the slug, as D7 states. F7's open question narrows to per-area collapsible blocks inside each project; D7's per-section Tasks, Notes and Done folds stand until the parent answers it.

Rationale: The board entry itself records that bodies render to design-<slug>.md (project-design-link, ffb53ae1) and that README.md's project sections stay without the body (project-body-in-render, 9cf88e2a), so the README holds no task or note body to fold.

### Added after Discussion (arbiter, 2026-11-17): No launch.json; the window title line's premise is stale

Decision: lym writes no launch.json, as the entry's Was ide-prime-setup part asks and Scope Out states. The entry's note that internal/vscode/config.go writes the bare slug as the window title no longer holds: since b0e9f535 the VS Code window title is boardengine.ConsoleWindowTitle, the same title as the worktree's terminal window. F8's open question narrows to whether the entry's LY: <name> title form and the prime's folderOpen chain (reed up then reed attach), both reversed by landed work, are still wanted.

Rationale: The entry says No launch.json outright. b0e9f535 merged IDEWindowTitle and TerminalWindowTitle into ConsoleWindowTitle so both windows show one title, and 6b037083 (closes #633) dropped every folderOpen chain so that opening a worktree starts nothing; reinstating either reverses a landed design, which the entry does not say it wants.

### Added after Discussion (parent, 2026-11-17): The phone goal is met by today's README

Decision: The operator said in chat (2026-11-17) that today's README works on the phone. The task builds no per-area collapsible blocks and no new README folds for the phone's sake; D7's collapsible sections stay only if another goal of the entry needs them, and the entry's phone line is dropped from Open risks.

Rationale: The operator judged the phone view good enough as it is, so changing the README layout for it would be work nobody asked for.

### Added after Discussion (parent, 2026-11-17): The prime's open chain and LY window title stay out

Decision: The operator confirmed in chat (2026-11-17) that the prime's folderOpen chain (reed up then reed attach) and the LY: <name> window title stay out of this task, as the record's Scope Out says.

Rationale: Both were changed on purpose after the old ide-prime-setup note was written (6b037083 dropped the open chain, b0e9f535 set the console window title), and the operator does not want them back.

### Added after Discussion (arbiter, 2026-11-17): D6's workspace seam keeps the workspace schema in internal/vscode

Decision: D6's third bullet as it now stands settles the round-2 fabric-boundary finding, and the task builds it as written. fabricengine.RegenerateHubWorkspace calls the installed workspace-content hook, reads only the returned members' names to refuse its own lym.generated mark, and hands the members on unread to internal/vscode. The workspace builder behind vscode.BuildWorkspace splices them after settings and refuses a member named folders or settings. Fabric names no VS Code workspace key besides the mark it already owns, and gains no knowledge of tasks, lym verbs or the lym binary. Do not move the splice or the folders and settings refusal back into fabric, and do not move the mark refusal into internal/vscode.

Rationale: PATTERN-fabric-boundary says that a change touching fabric's existing internal/vscode import moves knowledge out and never extends it. Round 2's F2 fix asked for exactly this placement, and the round-2 fixer made it. In internal/fabricengine/hubworkspace.go, fabric already leaves the file's shape to vscode.BuildWorkspace and owns beamWorkspaceMark. Passing opaque members through that call adds no workspace-schema knowledge to fabric.

### Added after Discussion (arbiter, 2026-11-19): board.html and board-data.js stay committed render outputs

Decision: The two files stay render outputs of every board write, listed in the render manifest, committed by the sync and the owner-set write, and claimed by FabricReplay, as D1 to D3 state. They serve the read-only page opened at file://, beside the server's editable page over http. Do not drop them or narrow D1 to D3 for them. Remove the Open risks bullet that asks the parent this question (the section then states that no risk is open), and change the support log's question-ledger entry Q8 and the Rejected alternatives bullet 'Drop board-data.js and board.html as render outputs' so they cite this decision instead of 'left to the parent in Open risks'.

Rationale: The board entry's Delivers line names both files as render outputs of every board write, committed by the sync and the owner-set write and claimed by FabricReplay. The operator's 'Wanted next (operator, 2026-11-19)' section adds editing and live refresh and revokes no deliverable, so dropping the files would narrow a deliverable the entry names, which is not the Discussion's call. Keeping them is what the entry asks and what the record already designs.

### Added after Discussion (arbiter, 2026-11-19): open's refusal naming lym board rerender is dropped; the rerender line lives in the file:// page

Decision: lym board open has no refusal for a missing board.html: D5 stands as written, and the open verb never reads the Bolt's board.html or board-data.js. The entry's 'lym board rerender' way forward stays where its condition still occurs: D4's last bullet, the line a page opened at file:// shows when board-data.js is missing or did not parse, with its D4 criterion. Remove the Open risks bullet that asks the parent to confirm this, and change the support log's question-ledger entry Q9 so it cites this decision instead of 'raised to the parent in Open risks'.

Rationale: The Delivers line's refusal guarded the old design, in which open handed the browser the committed board.html. The operator's 'Wanted next (operator, 2026-11-19)' section asks for writes from the page, which a file:// page cannot send (support log Turn 2), so open now opens the server's page, built from the embedded asset and a live projection (D2, D5). A refusal on a missing board.html would then stop open for a file it never reads, a refusal with no failure behind it. Dropping it removes no capability the entry asks for and widens nothing, and the rerender way forward still reaches the operator where a missing board-data.js does break a page.

### Added after Discussion (parent, 2026-11-19): Open lym board open refusal dropped; committed render outputs stay

Decision: Confirmed: lym board open drops its refusal naming lym board rerender, since the server serves the embedded page. board.html and board-data.js stay committed render outputs beside README.md, serving the read-only page opened by hand at file://.

Rationale: The operator agreed on 2026-11-19 that the arbiter keeps board.html and board-data.js; with the server serving the page, a missing board.html in the Bolt no longer has to stop the verb.

### Added after Discussion (parent, 2026-11-19): Opener waited on through cmd.Wait, not proc.Reap

Decision: The opener process is waited on through cmd.Wait in one goroutine rather than proc.Reap; card 6's startOpener bullet and the Shared Decisions bullet stand.

Rationale: D5 requires a failed opener to return its error, which needs the exit status proc.Reap discards; PATTERN-child-processes' reap-children rule admits cmd.Wait.

## Constraints

- `PATTERN-cwd-resolution`: nothing is linked or junctioned into a worktree; the hub path is resolved in lym.
- `PATTERN-fabric-boundary`: `fabricengine` gains no knowledge of tasks, verbs or the lym binary;
  `beam start` reaches the workspace files only through the general workspace-content seam of D6.
  Fabric's existing `internal/vscode` import is not extended: the workspace file's key knowledge stays in `internal/vscode`.
- `PATTERN-child-processes`: the opener spawn and the server spawn go through `internal/logger`, detached with `proc.Detach` and reaped with `proc.Reap`;
  the opener is waited on within D5's bound;
  the server registers in `internal/runproc` under `runproc.HubOwner` in its own role, and its binary is named through `selfexec.LymBinary`.
- `PATTERN-daemon-wakeups`: the server's watch loop is an unbounded wait; it keeps the one-second floor, uses `internal/fswatch` as its event source,
  and joins the background's list and the declared list in `cmd/lym/daemonwakeups_test.go` in the same commit.
  The long poll of `GET /api/board` is bounded under a minute and is outside the rule.
  `open`'s wait on the opener, for a caller with a strand name, takes the keep-warm cap (D5).
- `PATTERN-lym-files`: the server's state, lock, launcher and log live under the hub's board-level `.lym`, in a directory a `lymdirs` constant names; the lock path comes from `internal/lock`.
- `PATTERN-cli-cobra`: `lym board open` and `lym board serve` are subcommands in the existing board cobra subtree.
- `PATTERN-refusal-way-forward`: each refusal of `open` and `serve` names its way forward, and a test reaches it.
- `PATTERN-tolerant-records`: the server's state file is a Go-written JSON record, read tolerant of unknown keys and written through `internal/state`.
- `PATTERN-code-conventions`: `board.html`, `board-data.js`, the task labels, the state directory and the route paths are named constants;
  the platform opener and the Windows owner-only DACL (D9) live in build-tagged files, and no language, engine or branch is named.
- `PATTERN-test-only-code`: the opener seam that tests substitute is reached from production wiring.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: the server's handlers are tested in-process through a loopback test server, at the lowest tier that fits;
  the spawn of `lym board serve` by `open` is tested at the tier that allows a built `lym`.
- Lym's keys avoid `alt+shift+1` to `alt+shift+6`, which mill's own keybindings block holds.
- The test suite has no JavaScript runtime (`go.mod` carries none), so the viewer's in-browser behaviour is a review check;
  the Go side pins what the page is handed and what the server accepts and refuses.

## Auto-mode assumptions

- D4: the viewer groups like the README, not by the demo's Tasks/Bugs/Enhancements.
- D4: ages are computed against the browser's clock.
- D6: task labels `board open` and `beam start`; keys `alt+shift+9` and `alt+shift+0`.
- D8: a derived hash is the revision (ledger Q2).
- D9 to D11: editing and live refresh go through an on-demand loopback server (ledger Q1, Q3, Q4, Q6).
- D10: the page's edit fields (ledger Q5).
- D12: `open` and `serve` refuse in a run agent's session (ledger Q7).

## Open risks

- The entry's Delivers line names `lym board open` with "its refusal naming lym board rerender", which D5 drops:
  the server serves the embedded page, so a missing `board.html` in the Bolt no longer stops the verb.
  With the server in place, the committed `board.html` and `board-data.js` serve only a page opened by hand at `file://`.
  For the run's parent: confirm the dropped refusal, and say whether the two committed render outputs should stay now that editing needs the server.

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - board-html-not-pinned-to-lf (class design, severity MEDIUM, open in rounds 1)
  - page-busy-loops-on-absent-board-json (class design, severity MEDIUM, open in rounds 1)
  - verb-wiring-behaviors-untested (class design, severity MEDIUM, open in rounds 1)
  - watch-loop-no-one-second-floor (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D1: a render's file map holds `board-data.js`; stripping its `window.BOARD = ` prefix and `;` suffix leaves JSON that decodes,
  and each task's or note's section, project handle and waited-on handles match the README's placement of that entry.
- D1: the projection lists every open project with its parent project's handle, so a sub-project names its parent.
- D1: a body holding `</script>` and `<b>` appears in `board-data.js` with no literal `</script>` or `<b>`.
- D1: type and area labels are split by the board's vocabulary, so a type label never appears among an entry's area labels.
- D1: each entry's revision in the projection equals the revision D8's function computes for that entry,
  and the server's `GET /api/board` returns, render time aside, the projection the same function builds over the same store and config.
- D2: a render's `board.html` equals the embedded asset byte for byte, and a second render of a changed board leaves its bytes unchanged; the server's `GET /` returns the same bytes.
- D2: the embedded `board.html` references `board-data.js` through a script tag and holds no `http://` or `https://` resource URL.
- D3: `RenderToDisk` writes both files and its manifest lists them; a board write's sync commit holds both beside `README.md` when they changed.
- D3: `FabricReplay.Claims` claims `board-data.js` and `board.html`.
- D4: Review: the viewer filters by kind, type and area with OR inside a group and AND across groups, searches text case-insensitively,
  nests projects and sections like the README with counts, expands an entry to its brief, facts and body, escapes HTML in bodies,
  renders a title, brief or refusal message holding an HTML tag as literal text, links design and decision files by a `./` relative path at `file://` and by the file route over `http`,
  keeps only `http`, `https` and scheme-less relative links, renders `javascript:`, `data:`, `//host` and `/\host` targets as plain text,
  follows the colour scheme, shows no edit control at `file://`, shows the `lym board rerender` line when the data is missing at `file://`,
  and retries a failed request before showing the `lym board open` line.
- D5: `lym board open` run from a worktree's anchored directory, with the server's lock held by a live server, starts no server and hands the opener seam a launcher path under the server's state directory.
- D5: an `open` during a starting server's build, with the lock held and the previous server's state file left, hands over the new server's address;
  an `open` while a server drains on exit starts a fresh server.
- D5: `open` run from a task pair's worktree starts a server whose working directory is the prime's anchored directory.
- D5: the envelope's address holds no token.
- D5: with the lock held and no state that answers, `open` refuses naming the server log, and starts nothing.
- D5: with a strand name set, the keep-warm cap injected, and an opener that does not exit, `open` returns success at the cap.
- D5: `lym board open` with the lock free starts a server even when a leftover state file names a port, waits for the new server to answer with its token, and then hands the opener the launcher; with a server that never answers it refuses naming the server log.
- D5: `lym board open` under `--board-path` refuses naming a run from a worktree of the hub, and starts no process.
- D5: an opener that fails surfaces its error naming the opener.
- D6: seeding the prime, a beam and a task pair each writes `reed attach` and `board open` into the worktree's `tasks.json`, and none of them writes `beam start` there.
- D6: the `board open` task's presentation sets `"reveal": "always"` and `"focus": false`.
- D6: the hub workspace file holds one `beam start` task running `lym beam start root`, and a beam's workspace file holds one running `lym beam start <beam>`.
- D6: a regeneration with no workspace-content hook installed writes workspace files with no `tasks` key.
- D6: a regeneration whose hook returns a member named `lym.generated`, `folders` or `settings` returns an error and writes no workspace file.
- D6: the keybindings block holds `alt+shift+0` running `beam start` and `alt+shift+9` running `board open`, and a second seed leaves exactly one of each.
- D8: an edit carrying the entry's current revision is saved; one carrying an older revision, including one older only by a status change, is refused with the fresh entry, and `board.json`, the README and the Bolt's git state are unchanged.
- D8: on a `board.json` holding an entry the load migrations change, the revision the server's projection gives that entry is accepted by an edit.
- D8: an edit carrying the revision a successful write returned is saved.
- D8: an edit of an entry another write removed is refused as stale with no entry; a create of an existing slug is refused with the existing entry.
- D8: the existing CLI write verbs behave as before, with no revision.
- D9: a request whose `Host` is not `127.0.0.1:<port>`, an `/api` request without the token or with a wrong one, a write without `Content-Type: application/json`, a write whose `Origin` is another origin, and a body above the cap are each refused and change nothing.
- D9: no response carries an `Access-Control-Allow-Origin` header; every response carries `Cache-Control: no-store` and `X-Content-Type-Options: nosniff`; the page's `Content-Security-Policy` holds no `'unsafe-inline'`.
- D9: `GET /file/<name>` without `t` or with a wrong `t` is refused and serves nothing.
- D9: the state file and the launcher are readable by the user alone, on Windows each file's DACL grants only the owner,
  the launcher's address uses `127.0.0.1`, and the opener's argument holds no token.
- D10: each operation, create, edit, release, promote, draft and undraft, applied through `POST /api/write` leaves `board.json`, its date stamps aside, as the matching CLI verb leaves it, and answers with the fresh projection.
- D10: draft of a task whose status is not none, and undraft of a task whose status is not `draft`, are refused and change nothing.
- D10: release of a task in a run status is refused naming `lym board set-status`, and changes nothing.
- D10: Review: the page shows the create form and the edit form, offers only the release, promote, draft and undraft actions an entry's kind and status allow,
  and on a stale refusal shows the fresh entry beside the unsent edit, which stays in its form.
- D10: an edit of a running task's scope through the server is refused with the run lock's message, as the CLI refuses it.
- D10: `GET /api/board` with the current board revision returns once another writer changes `board.json`, with the new projection; with no change it answers "unchanged" at its bound.
- D10: `GET /file/<name>` serves a name the projection lists and answers not found for any other, a path with a separator included.
- D11: a second `lym board serve` for the same hub exits at once with success after its lock wait while the first holds the lock; `lym board serve` under `--board-path` refuses.
- D11: the server exits when no `/api` request arrives within its idle bound, and when its binary is replaced, using an injected clock and replacement check, and a write in flight at that moment completes.
- D11: with the file watch unavailable, a change to `board.json` still wakes a waiting `GET /api/board` through the timer.
- D12: `lym board open` and `lym board serve` with a run agent's strand name refuse and start nothing; with an orch's strand name or none they proceed.
- D13: Review: the HTTP server, its checks, its state and its watch loop live in `internal/boardserve` (new), which writes the board only through `boardengine`'s exported methods.

## Notes for the plan writer

- The Added after Discussion entries name D7, the collapsible README sections of an earlier draft that the parent's phone answer dropped; this record leaves the id D7 unused.
- The README section names and order already live in the render code that builds `layerSection` and the Notes sections; the projection reads them from there.
- `fabricengine.RegenerateHubWorkspace` writes the hub's and every beam's workspace file, rewriting each only when its bytes change.
- D8's check goes in `boardCriticalSection` in `internal/boardengine/board.go`.
