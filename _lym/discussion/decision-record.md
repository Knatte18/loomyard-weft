# Decision record — board-viewer

## Goal

Give the operator an interactive view of the board, with filters, opened from VS Code by one key instead of a typed lym command,
and make the README easier to read on a phone in the GitHub app.

## Scope

In:

- `board-data.js` and `board.html` as render outputs of the board, written, committed and replayed with the README.
- The viewer page: filters, free-text search, grouping, expandable entries, light and dark.
- A `lym board open` verb that opens the hub's viewer in the default browser.
- VS Code tasks `board open` and `beam start`, and two key bindings, written by `lym ide seed` and `lym ide spawn`.
- Collapsible Tasks, Notes and Done sections in every rendered board markdown file.

Out:

- Window titles, the folderOpen chain and `launch.json` from the entry's `Was ide-prime-setup` part:
  console titles landed in `b0e9f535`, commit `6b037083` dropped the folderOpen chain, and nothing writes `launch.json`.
- A Live Preview panel inside VS Code: the task opens the default browser only.
- macOS: lym's platform files cover Linux and Windows only, as `internal/vscode/launch_linux.go` and `internal/vscode/launch_windows.go` show.

## Decisions

### D1 — `board-data.js` is a render output built from the README's own model

`boardengine.Render` returns `board-data.js` beside the README in its file map.
Its content is `window.BOARD = <json>;` and a newline.
The JSON is a view projection computed by the same code that builds the README, so the viewer recomputes nothing:

- top level: the render time, the prime owner, the configured type-label order, a project list and an entry list;
- per project, the root `P0` included: handle, slug, title, brief, labels, the parent project's handle, and its design file name;
- per task or note: handle, slug, title, kind, type labels and area labels split by the board's vocabulary, status, its project's handle,
  the README section it sits in (Running, Ready, a layer, Independent, Draft, a Notes type, Done),
  the handles it waits on, brief, body, owner, recipe, created and revised timestamps,
  and the design and decision file names when they exist.

Timestamps are absolute; the viewer computes ages against the browser's clock.
The JSON is encoded with Go's default HTML escaping, so a `<`, `>` or `&` in any field is written as a `<`-style escape
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
- `boardengine.FabricReplay.Claims` claims both names, as it claims `README.md`, so a Bolt replay re-renders them instead of dropping them as unclaimed.

Bound: these paths carry the README's own exposure to mixed lym versions on one hub.
A machine on an older lym leaves `board-data.js` stale until a newer one writes;
a downgrade removes both files through the manifest and the sync commits the removal;
two binaries with different viewers rewrite `board.html` on each write;
and a replayed commit that changed only rendered files is replayed as empty, so its new `board.html` waits for the next board write.
Each heals on the next write by an up-to-date binary.

### D4 — the viewer page

- Filters: kind (task, note), type labels and area labels, each a set of toggles.
  Toggles in one group are OR'd, groups are AND'd, and an empty group filters nothing.
- Free-text search, case-insensitive, over handle, slug, title, brief and body.
- Grouping mirrors the README: each project a group, a sub-project nested under its parent, and a project's own tasks and notes grouped by section in the README's order.
  Each group shows its count after filtering, and an empty group is hidden.
- Each entry is a row with handle, title, status and labels; clicking it expands the brief, the facts line and the body.
  A discussed entry links to its `decision-<slug>.md` and every entry to its design file, by relative path.
- The body renders a Markdown subset: headings, paragraphs, bullet and numbered lists, fenced code, code spans, bold, italic and links.
  Every body is HTML-escaped before the subset applies, and a link is kept only when its target is `http`, `https` or a relative path.
- Light and dark follow `prefers-color-scheme`.
- A header shows the render time and the prime owner.
- When `window.BOARD` is undefined, because `board-data.js` is missing or did not parse, the page shows one line naming `lym board rerender`.

### D5 — `lym board open` opens the viewer

A new operator verb `lym board open` opens the hub's `board.html` in the default browser.

- It resolves its cwd with `lymcwd.ResolveWithin`, as a read-only verb may, so it runs from anywhere in a worktree.
  It reads the board directory as the other board verbs do, so the board subtree's `--board-path` flag points it at that directory's `board.html`;
  it takes no argument of its own and opens no other file.
- When `board.html` is absent it refuses, naming `lym board rerender` as the way forward, and starts nothing.
  `lym board rerender` renders and syncs, so the file then exists and is committed.
- The opener is a per-platform build file: `xdg-open <path>` on Linux, `rundll32 url.dll,FileProtocolHandler <path>` on Windows.
  The spawn is logged and waited on.
- A failed opener, such as a Linux machine without `xdg-open`, returns its error with the opener named.
- On success it prints the opened path in its JSON envelope.

Bound: `xdg-open` with no desktop detected runs a browser in the foreground, and the verb then waits until that browser exits.

Rationale: lym resolves the hub path, so nothing is linked into a worktree;
the same verb serves a terminal and the VS Code task.

### D6 — VS Code tasks and key bindings

- `board open`, running the absolute lym binary with `board open`, goes into every worktree's `.vscode/tasks.json` beside `reed attach`;
  its presentation does not reveal or focus the terminal.
- `beam start` goes into the workspace files' `tasks` key, never into a folder's `tasks.json`, so a multi-root window holds exactly one:
  the hub's `<RepoName>.code-workspace` runs `lym beam start root` and each beam's `<RepoName>-<beam>.code-workspace` runs `lym beam start <beam>`,
  each with the prime's or that beam's anchor as its working directory.
  It is presented like `reed attach`: revealed, focused, in a new panel.
  The workspace files are built by the existing builder in `internal/vscode/workspace.go` and written by `fabricengine.RegenerateHubWorkspace`.
- The `// lym:begin` … `// lym:end` block in the user `keybindings.json`, written by the existing writer in `internal/vscode/keybindings.go`, gains two entries:
  `alt+shift+0` runs the task `beam start` and `alt+shift+9` runs `board open`, through `workbench.action.tasks.runTask`.
  Every seed or spawn rewrites them with the block.
- Task labels follow the existing `reed attach` style: the verb's words after `lym`.

Bound: the key bindings are user-global, so a key can name a task its window lacks, and VS Code then reports the missing task.
That happens for `alt+shift+0` in a task pair's bare-folder window, in a prime or beam window opened as a bare folder because the prime did not resolve,
in any window whose worktree has a git-tracked `.vscode/tasks.json`, which seeding skips today,
and in a window seeded before the upgrade until it is seeded again.
In a multi-root window several folders carry `board open`; every copy runs the same verb on the same hub, so whichever copy VS Code runs, the result is the same.

Rationale: tasks are the mechanism the `reed attach` binding already uses;
a workspace-level `beam start` with an explicit slug starts the beam the window belongs to.

### D7 — collapsible README sections

In every rendered board markdown file (`README.md` and every `project-<slug>.md`), each project's Tasks, Notes and Done sections become `<details>` blocks
with a `<summary>` holding the section name and count in bold.
Tasks is open by default; Notes and Done are closed.
The layer and note-type subsections inside stay headings.
The README keeps its grouping by dependency layer and note type, and bodies stay in `design-<slug>.md`, linked from the slug as today.
Area grouping is the viewer's area filter (D4), not a README grouping.

Rationale: the entry asks for `<details>` blocks for the GitHub app, and the README already wraps each project in one;
folding Notes and Done shortens the phone scroll to the running and ready work.

## Constraints

- `PATTERN-cwd-resolution`: nothing is linked or junctioned into a worktree; the hub path is resolved in lym.
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
- D7: Notes and Done fold closed; Tasks stays open.

## Open risks

- The entry's phone part asks for README sections grouped by type and area, with collapsible blocks per area and per body.
  Landed work since then (`board-readme-per-project`, `af86384e`, `d54f5181`, `a99b3b4d`, `ffb53ae1`, `9cf88e2a`) groups tasks by dependency layer,
  notes by type, and moves bodies to `design-<slug>.md`.
  D7 builds collapsible sections over that structure; the parent confirms this reading of the phone goal, or asks for per-area grouping inside each project.
- The entry's `Was ide-prime-setup` part is largely outdated by the landed work Scope names.
  The record keeps only its tasks and keys; the parent confirms nothing else in that part is still wanted.

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
  nests projects and sections like the README with counts, expands an entry to its brief, facts and body, escapes HTML in bodies, keeps only `http`, `https` and relative links,
  follows the colour scheme, and shows the `lym board rerender` line when the data is missing.
- D5: `lym board open` with no `board.html` refuses naming `lym board rerender` and starts no process.
- D5: `lym board open` run from a subdirectory of a worktree with `board.html` present hands the opener seam the absolute path of the hub's `board.html` and reports it.
- D5: an opener that fails surfaces its error naming the opener.
- D6: seeding the prime, a beam and a task pair each writes `reed attach` and `board open` into the worktree's `tasks.json`, and none of them writes `beam start` there.
- D6: the hub workspace file holds one `beam start` task running `lym beam start root`, and a beam's workspace file holds one running `lym beam start <beam>`.
- D6: the keybindings block holds `alt+shift+0` running `beam start` and `alt+shift+9` running `board open`, and a second seed leaves exactly one of each.
- D7: a rendered project section wraps Tasks in `<details open>` and Notes and Done in closed `<details>`, each summary naming the section and its count.

## Notes for the plan writer

- The README section names and order already live in the render code that builds `layerSection` and the Notes sections; the projection reads them from there.
- `fabricengine.RegenerateHubWorkspace` writes the hub's and every beam's workspace file, rewriting each only when its bytes change;
  every caller of it, not only `lym ide seed`, then writes the `beam start` tasks, so the tasks' content must come from inputs every caller has.
