# Decision record — board-viewer

## Goal

Give the operator an interactive view of the board, with filters, opened from VS Code by one key instead of a typed lym command.

## Scope

In:

- `board-data.js` and `board.html` as render outputs of the board, written, committed and replayed with the README.
- The viewer page: filters, free-text search, grouping, expandable entries, light and dark.
- A `lym board open` verb that opens the hub's viewer in the default browser.
- VS Code tasks `board open` and `beam start`, and two key bindings:
  `lym ide seed` and `lym ide spawn` write `board open` and the keys,
  and the hub workspace regeneration writes `beam start` through the ide module's workspace-content hook.

Out:

- Window titles, the folderOpen chain and `launch.json` from the entry's `Was ide-prime-setup` part.
- A Live Preview panel inside VS Code: the task opens the default browser only.
- macOS: lym's platform files cover Linux and Windows only, as `internal/vscode/launch_linux.go` and `internal/vscode/launch_windows.go` show.

## Decisions

### D1 — `board-data.js` is a render output built from the README's own model

`boardengine.Render` returns `board-data.js` beside the README in its file map.
Its content is `window.BOARD = <json>;` and a newline.
The JSON is a view projection computed by the same code that builds the README, so the viewer recomputes nothing:

- top level: the render time, the prime owner, the configured type-label order,
  the README's section names in the README's order, a project list and an entry list;
- per project, the root `P0` included: handle, slug, title, brief, labels, the parent project's handle, and its design file name;
- per task or note: handle, slug, title, kind, type labels and area labels split by the board's vocabulary, status, its project's handle,
  the README section it sits in (Running, Ready, a layer, Independent, Draft, a Notes type, Done),
  the handles it waits on, brief, body, owner, recipe, created and revised timestamps,
  and the design and decision file names when they exist.

Timestamps are absolute; the viewer computes ages against the browser's clock.
The JSON is encoded with Go's default HTML escaping, so a `<`, `>` or `&` in any field is written as a JSON unicode escape, a backslash and `u` followed by the hex code `003c`, `003e` or `0026`,
and a body holding `</script>` cannot end the script.

Rationale: a projection keeps layering, handles, sections and project nesting in Go, where the README already computes them,
so the viewer and the README never disagree on where an entry sits.

### D2 — `board.html` is an embedded asset written as a fixed render output

The viewer is one self-contained HTML file with inline CSS and JavaScript, embedded in the binary with `go:embed` in `internal/boardengine`.
`boardengine.Render` returns it verbatim as `board.html`.
Rewriting identical bytes leaves no git change, so with one lym version on a hub the file changes in the Bolt only when a new binary ships a changed viewer.
The page loads its data only through `<script src="board-data.js">`, since a browser blocks `fetch` from a page opened at `file://`.
It loads nothing from the network.

Both file names are named constants in `internal/boardengine`.

Rationale: a render output needs no separate install step, survives a fresh Bolt clone, and is removed and committed through the existing render manifest and sync path.

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
A downgrade removes both files and commits the removal,
and since the older binary claims neither file, that commit replays as a plain cherry-pick rather than through the by-entry merge,
and it conflicts whenever a newer binary has written `board-data.js` since.
A conflicted replay restores the starting tip, and the board write is skipped as a diverged Bolt with the existing records-conflict way forward,
which names the conflicting paths;
the case recurs on each board write by the older binary until that machine is upgraded.

### D4 — the viewer page

- Filters: kind (task, note), type labels and area labels, each a set of toggles.
  Toggles in one group are OR'd, groups are AND'd, and an empty group filters nothing.
- Free-text search, case-insensitive, over handle, slug, title, brief and body.
- Grouping mirrors the README: each project a group, a sub-project nested under its parent,
  and a project's own tasks and notes grouped by section in the order D1's section list gives.
  Each group shows its count after filtering, and an empty group is hidden.
- Each entry is a row with handle, title, status and labels; clicking it expands the brief, the facts line and the body.
  A discussed entry links to its `decision-<slug>.md`, and an entry to its design file when the data names one, by relative path.
- The body renders a Markdown subset: headings, paragraphs, bullet and numbered lists, fenced code, code spans, bold, italic and links.
  Every body is HTML-escaped before the subset applies;
  the escape covers double quotes, so an escaped target cannot leave its `href` attribute.
  A link's target first has leading whitespace and control characters trimmed,
  and every tab, carriage return and line feed removed, as a browser's URL parser does.
  It is kept only when it then has the scheme `http` or `https`, compared case-insensitively,
  or has no scheme and does not open with two characters that are each a slash or a backslash.
  Every other target, `javascript:`, `data:`, `//host` and `/\host` among them, renders as plain text.
- Light and dark follow `prefers-color-scheme`.
- A header shows the render time and the prime owner.
- When `window.BOARD` is undefined, because `board-data.js` is missing or did not parse, the page shows one line naming `lym board rerender`.

### D5 — `lym board open` opens the viewer

A new operator verb `lym board open` opens the hub's `board.html` in the default browser.

- The board subtree's `PersistentPreRunE` (`internal/boardcli/cli.go`) resolves the `open` verb's cwd with `lymcwd.ResolveWithin`
  instead of the `lymcwd.Resolve` it runs for the other board verbs, as a read-only verb may,
  so the verb runs from anywhere inside the worktree's anchored directory or its records sibling.
  Without the subtree's `--board-path` flag, the board directory comes from the resolved layout, whose board config it loads as for the other board verbs.
  `--board-path` bypasses resolution and config, and the verb opens `board.html` in that directory.
  The verb takes no argument of its own and opens no other file.
- When `board.html` is absent it refuses and starts nothing.
  The refusal names `lym board rerender` as the way forward and the worktree's anchored directory to run it from,
  since that verb keeps the strict resolution.
  Under `--board-path` no worktree was resolved, so the refusal names `lym board rerender`, run from the anchored directory of a worktree of the hub.
  `lym board rerender` renders and syncs, so the file then exists and is committed.
- The opener is a per-platform build file: `xdg-open <path>` on Linux, `rundll32 url.dll,FileProtocolHandler <path>` on Windows.
  The spawn is logged and waited on.
- A failed opener, such as a Linux machine without `xdg-open`, returns its error with the opener named.
- On success it prints the opened path in its JSON envelope.

Bound: `xdg-open` with no desktop detected runs a browser in the foreground, and the verb then waits until that browser exits.
While it waits, the `board open` task stays active, and a second key press is answered by VS Code's running-task prompt.

Rationale: lym resolves the hub path, so nothing is linked into a worktree;
the same verb serves a terminal and the VS Code task.

### D6 — VS Code tasks and key bindings

- `board open`, running the lym binary with `board open`, goes into every worktree's `.vscode/tasks.json` beside `reed attach`,
  written by `vscode.WriteConfig` from `lym ide seed` and `lym ide spawn`.
  Its presentation sets `"reveal": "always"` and `"focus": false`:
  the terminal shows the verb's output, so D5's refusal and a failed opener's error are seen, and it does not take focus.
  The terminal panel therefore comes forward on every press, a successful one included.
- `beam start` goes into the workspace files' `tasks` key, never into a folder's `tasks.json`, so a multi-root window holds exactly one:
  the hub's `<RepoName>.code-workspace` runs `lym beam start root` and each beam's `<RepoName>-<beam>.code-workspace` runs `lym beam start <beam>`,
  each with the prime's or that beam's anchor as its working directory.
  It is presented like `reed attach`: revealed, focused, in a new panel.
- Fabric learns nothing of tasks, lym verbs or the lym binary.
  `fabricengine` offers a general workspace-content seam, installed process-wide as `InstallPairFolderNamer` is.
  The installed hook is handed the hub path, the beam slug a workspace file belongs to (empty for the hub's own file) and that worktree's anchor path,
  and returns extra top-level members as raw JSON keyed by name.
  `fabricengine.RegenerateHubWorkspace` reads only the members' names, to refuse one named as the beam file's generated mark, which fabric owns,
  and passes the members on to `internal/vscode`, whose workspace builder behind `vscode.BuildWorkspace` splices them verbatim after `settings`.
  Fabric so names no workspace key but its own mark, and every caller of the regeneration writes the members; with no hook installed a file carries no extra member.
  The builder refuses a member named as a key it writes itself (`folders` or `settings`).
  Either refusal is a programmer error the regeneration returns, writing nothing.
  `cmd/lym/main.go` installs the ide module's hook beside the pair folder namer;
  the hook builds the `tasks` member through `internal/vscode`.
- Both commands run the absolute path `selfexec.LymBinary` returns in the writing process, through `ideengine.LymBinary`,
  and fall back to bare `lym` when it errors, as `vscode.WriteConfig` does for `reed attach`.
- The `// lym:begin` … `// lym:end` block in the user `keybindings.json`, written by the existing writer in `internal/vscode/keybindings.go`, gains two entries:
  `alt+shift+0` runs the task `beam start` and `alt+shift+9` runs `board open`, through `workbench.action.tasks.runTask`.
  Every seed or spawn rewrites them with the block.
- Task labels follow the existing `reed attach` style: the verb's words after `lym`.

Bound: the key bindings are user-global, so a key can name a task its window lacks, and VS Code then reports the missing task.
That happens for `alt+shift+0` in a task pair's bare-folder window, in a prime or beam window opened as a bare folder because the prime did not resolve,
and in a window seeded before the upgrade until it is seeded again.
It happens for `alt+shift+9` in any window whose worktree has a git-tracked `.vscode/tasks.json`, which seeding skips today,
so that window lacks `board open` as it lacks `reed attach`,
and in a window seeded before the upgrade until it is seeded again.
A binary older than this change that regenerates the workspace files, or seeds or spawns,
rewrites them without the `beam start` tasks, the `board open` task and the two keys.
The next regeneration by an up-to-date binary restores the `beam start` tasks,
and its next seed or spawn restores `board open` and the keys, as D3 states for the board files.
In a multi-root window several folders carry `board open`; every copy runs the same verb on the same hub, so whichever copy VS Code runs, the result is the same.

Rationale: tasks are the mechanism the `reed attach` binding already uses;
a workspace-level `beam start` with an explicit slug starts the beam the window belongs to.

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

## Constraints

- `PATTERN-cwd-resolution`: nothing is linked or junctioned into a worktree; the hub path is resolved in lym.
- `PATTERN-fabric-boundary`: `fabricengine` gains no knowledge of tasks, verbs or the lym binary;
  `beam start` reaches the workspace files only through the general workspace-content seam of D6.
  Fabric's existing `internal/vscode` import is not extended: the workspace file's key knowledge stays in `internal/vscode`.
- `PATTERN-child-processes`: the opener spawn goes through `internal/logger` and is waited on.
- `PATTERN-cli-cobra`: `lym board open` is a subcommand in the existing board cobra subtree.
- `PATTERN-code-conventions`: `board.html`, `board-data.js` and the task labels are named constants;
  the platform opener lives in build-tagged files, and no language, engine or branch is named.
- `PATTERN-test-only-code`: the opener seam that tests substitute is reached from production wiring.
- Lym's keys avoid `alt+shift+1` to `alt+shift+6`, which mill's own keybindings block holds.
- The test suite has no JavaScript runtime (`go.mod` carries none), so the viewer's in-browser behaviour is a review check;
  the Go side pins what the page is handed.

## Auto-mode assumptions

- D4: the viewer groups like the README, not by the demo's Tasks/Bugs/Enhancements.
- D4: ages are computed against the browser's clock.
- D6: task labels `board open` and `beam start`; keys `alt+shift+9` and `alt+shift+0`.

## Open risks

## Acceptance criteria

- D1: a render's file map holds `board-data.js`; stripping its `window.BOARD = ` prefix and `;` suffix leaves JSON that decodes,
  and each task's or note's section, project handle and waited-on handles match the README's placement of that entry.
- D1: the projection lists every open project with its parent project's handle, so a sub-project names its parent.
- D1: a body holding `</script>` and `<b>` appears in `board-data.js` with no literal `</script>` or `<b>`.
- D1: type and area labels are split by the board's vocabulary, so a type label never appears among an entry's area labels.
- D2: a render's `board.html` equals the embedded asset byte for byte, and a second render of a changed board leaves its bytes unchanged.
- D2: the embedded `board.html` references `board-data.js` through a script tag and holds no `http://` or `https://` resource URL.
- D3: `RenderToDisk` writes both files and its manifest lists them; a board write's sync commit holds both beside `README.md` when they changed.
- D3: `FabricReplay.Claims` claims `board-data.js` and `board.html`.
- D4: Review: the viewer filters by kind, type and area with OR inside a group and AND across groups, searches text case-insensitively,
  nests projects and sections like the README with counts, expands an entry to its brief, facts and body, escapes HTML in bodies,
  keeps only `http`, `https` and scheme-less relative links, renders `javascript:`, `data:`, `//host` and `/\host` targets as plain text,
  follows the colour scheme, and shows the `lym board rerender` line when the data is missing.
- D5: `lym board open` with no `board.html` refuses naming `lym board rerender` and the anchored directory to run it from, and starts no process.
- D5: `lym board open` run from a subdirectory of a worktree's anchored directory with `board.html` present hands the opener seam the absolute path of the hub's `board.html` and reports it.
- D5: an opener that fails surfaces its error naming the opener.
- D6: seeding the prime, a beam and a task pair each writes `reed attach` and `board open` into the worktree's `tasks.json`, and none of them writes `beam start` there.
- D6: the `board open` task's presentation sets `"reveal": "always"` and `"focus": false`.
- D6: the hub workspace file holds one `beam start` task running `lym beam start root`, and a beam's workspace file holds one running `lym beam start <beam>`.
- D6: a regeneration with no workspace-content hook installed writes workspace files with no `tasks` key.
- D6: the keybindings block holds `alt+shift+0` running `beam start` and `alt+shift+9` running `board open`, and a second seed leaves exactly one of each.

## Notes for the plan writer

- The Added after Discussion entries name D7, the collapsible README sections of an earlier draft.
  It was dropped once the parent answered that today's README serves the phone, so the task changes no README layout.
- The README section names and order already live in the render code that builds `layerSection` and the Notes sections; the projection reads them from there.
- `fabricengine.RegenerateHubWorkspace` writes the hub's and every beam's workspace file, rewriting each only when its bytes change;
  every caller of it, not only `lym ide seed`, runs the installed workspace-content hook.
  The hook's inputs are settled: the hub path, the beam slug (empty for the hub's own file),
  the prime's or that beam's anchor, which the regeneration already reads through `lymcwd.NamedWorktree`,
  and the binary path the hook reads itself through `ideengine.LymBinary`.
