---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/04-board-server-routes.md:63
    summary: "A non-create op with an empty revision takes the create path. Card 2 reads an empty revision as a create, and card 4 passes edit's, promote's, release's, draft's and undraft's request revision through unchecked. So an `edit` with `revision: \"\"` on an absent slug creates the entry through `UpsertTaskReporting`, outside create's field list and its existing-slug pre-check, and a `promote` with an empty revision on an existing note gets create's `already exists` refusal."
    fix: "In card 4's `applyWrite` bullet, state that every op but `create` refuses an empty `revision` before any write, naming the op and the way forward (reload the board and act on what it holds now). Card 4's Bound then states that only `create` reaches the empty-revision path. Add a row to `TestServer`'s refused-and-changing-nothing list: an `edit` with an empty `revision` of an absent slug is refused and `board.json` keeps its bytes."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/07-vscode-tasks-and-keys.md:33
    summary: "Cards 7 and 9 add routing edges with no stated bound, which `loom-block-plan-cards.md` line 44 requires. Card 7 adds two user-global keys that run whatever task carries the label first in any window, including a task of that label in a repo's own git-tracked `tasks.json`, which seeding leaves alone. Card 9 installs a process-wide hook that puts a `process` task into every hub and beam workspace file a lym process regenerates. Neither card's Intent says what the change lets through or what bounds it, as cards 2, 4, 5, 6 and 8 do."
    fix: "Add a `Bound:` paragraph to card 7's Intent. It states that each key runs only through `workbench.action.tasks.runTask` with a fixed label. VS Code runs the first task of that label it finds, so a window whose tracked `tasks.json` defines its own `board open` or `beam start` runs that task. A window lacking the label opens the filtered task picker, as D6's bound states. Add a `Bound:` paragraph to card 9's Intent. It states that the hook adds one `process` task running the lym binary, or bare `lym`, with the fixed args `beam start <beam>` in that worktree's anchor. Fabric reads only the member names and refuses its own mark, and `vscode.BuildWorkspace` refuses `folders` and `settings` (card 8)."
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/05-board-server-lifecycle.md:61
    summary: "Run's exit step closes the listener and treats that as taking no new request, but Go's `http.Server` keeps serving further requests on established keep-alive connections until `Shutdown` runs. The page's open connection can therefore land a write after the lock is released, which breaks D11's order: stop taking new requests, then release the lock. `Shutdown` also returns the closed listener's error, which step 6's \"returns nil\" leaves unhandled."
    fix: "Reword step 6 of card 5's `Run` bullet as follows. On exit, call `http.Server.SetKeepAlivesEnabled(false)` and close the listener, so neither a new connection nor a kept-alive one takes a request. Release the lock, call `beginDrain`, then wait for the requests in flight through `http.Server.Shutdown`, ignoring its error for the already-closed listener. Alternatively, start `Shutdown` in a goroutine before releasing the lock and wait on it after `beginDrain`."
  - id: F4
    severity: LOW
    class: decision
    origin: handler
    location: _lym/plan/00-overview.md:34
    summary: "D5 and the record's `PATTERN-child-processes` constraint both say the opener is reaped with `proc.Reap`. The plan waits on it through `cmd.Wait` in one goroutine instead, and records the change only in its Shared Decisions. The substitution is sound: the pattern's Reap-children rule admits `cmd.Wait`, and D5's requirement that a failed opener return its error needs the exit status `proc.Reap` discards. Still, it departs from the answer key's text with no Added after Discussion entry recording it."
    fix: "Ask the parent to record the substitution with `lym loom decision add`. The entry says the opener is waited on through `cmd.Wait` in one goroutine rather than `proc.Reap`, because D5's failed-opener error needs the exit status. Keep card 6's `startOpener` bullet and the Shared Decisions bullet as they are."
  - id: F5
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/06-board-serve-open-verbs.md:114
    summary: "D11's acceptance criterion says a second `lym board serve` exits at once with success while the first holds the lock. Only `Run`'s `ErrRunning` return is tested (card 5's `TestRun`). No test reaches the `serve` verb's mapping of `ErrRunning` to `{\"ok\":true,\"running\":true}` and exit 0, and `open`'s wait relies on that mapping to find the winner."
    fix: "Add one step to card 6's `TestBoardServeScenario`. After the first `open` starts the server, run `lym board serve` from the prime's anchor with the same extra environment, and assert it exits 0 with `running: true` in its envelope while the first server's state still answers `Probe`."
  - id: F6
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/02-entry-revision-check.md:49
    summary: "Two rows of `TestExpectingRevision` cover one behaviour: an edit carrying the revision `Board.View` gives is saved. The fourth row, a revision from the `Board.View` after a successful write, repeats the first, since `Board.View` reads the saved `board.json` either way. The D8 criterion it maps to, a revision that a successful write returned, is the server's answer, which card 4's `TestServer` row already pins."
    fix: "Cut the fourth bullet of card 2's `TestExpectingRevision` list (the edit carrying the revision a successful write's following `Board.View` returned). Leave the D8 criterion to card 4's `TestServer` row, the edit carrying the revision of the entry a successful write answered."
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/01-board-view-projection.md:41
    summary: "Two comments that list the board's render outputs and the replay's recognised files go stale, and no card names them. The `render.go` file header lists README.md, the project files and the design files as Render's outputs, but cards 1 and 3 add `board-data.js` and `board.html`. The comment over the replay's file-name constants in `replay.go` lists what a replay recognises at the Bolt root, but card 3 makes `Claims` claim the two new files."
    fix: "Add an Intent bullet to card 1 saying `render.go`'s file header names `board-data.js` among Render's outputs. Add one to card 3 saying the header also names `board.html`, and that the comment over `replay.go`'s file-name constants names both claimed files. Keep the fix in Intent text: a `render.go` or `replay.go` file target beside the cards' member targets in those files is refused by `redundant-file-target`."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/04-board-server-routes.md:36
    summary: "`docs/overview.md`'s package tree gives every `internal/` package one line, but no card adds the new `internal/boardserve/`. Cards 7 and 9 edit only that file's VS Code passages."
    fix: "Add `**Extends:**` with `- docs/overview.md` (backtick-wrapped) to card 4, before `**Uses:**`. Add an Intent sentence saying the package tree gains an `internal/boardserve/` line beside `internal/boardcli/` and `internal/boardengine/`, describing the board's loopback server. Add `**ImpactSummary:** a new package with no existing caller; docs/overview.md gains one package-tree line.` before `**Commit:**`. This was checked with `lym loom validate-plan --overlay`."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/06-board-serve-open-verbs.md:30
    summary: "Card 6 lists `internal/runproc/doc.go` under Extends, but that doc closes its list of the HubOwner roles with the sentence that they are `RoleSync` and `RoleSlackRelay`. Naming `RoleBoardServer` among the hub's roles rewrites that sentence, which Extends does not allow."
    fix: "Move `internal/runproc/doc.go` from card 6's `**Extends:**` group to its `**Edit:**` group, after `internal/runproc#TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`. This was checked with `lym loom validate-plan --overlay`."
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/00-overview.md:31
    summary: "The Shared Decisions bullet says an edit and a promote carry their revision straight to the write section. Card 4 refuses a promote of a task before the write, by reading the current `Board.View()`, because `promoteTo` returns an existing task unchanged without refusing."
    fix: "Reword the second line of the Shared Decisions revision bullet. An edit carries its revision straight to the write section. A promote first refuses a task by its kind in the current `Board.View()`, then carries its revision to the write section."
---

### [MEDIUM] A non-create op with an empty revision takes the create path

**Location:** `_lym/plan/04-board-server-routes.md:63`

**Issue:** Card 2 makes an empty expected revision mean "expects no entry of that slug: a create" (`02-entry-revision-check.md:31`).
Card 4's `applyWrite` hands `edit` and `promote` straight to `Expecting(slug, revision)` with no pre-check.
Nothing refuses an empty `revision` on those ops.
So an `edit` body with `"revision": ""` for an absent slug passes the write section and `UpsertTaskReporting` creates the entry.
That skips create's field list and the server's existing-slug pre-check.
Card 4's Bound, "the page reaches only these six writes", does not cover this path.
The page never sends such a request, so only a token holder reaches it.
Even so, it is an op the server's contract does not offer.

### [MEDIUM] Cards 7 and 9 state no bound for their new routing edges

**Location:** `_lym/plan/07-vscode-tasks-and-keys.md:33`, and `_lym/plan/09-beam-start-workspace-task.md:24`

**Issue:** The plan-writer block `loom-block-plan-cards.md` line 44 requires a card that introduces a routing edge to state, in its Intent, what the change lets through and what bounds it.
Card 7's keys are user-global and run a task by label through `workbench.action.tasks.runTask`.
VS Code runs the first task of that label in the window's folders.
A repo's tracked `.vscode/tasks.json`, which seeding leaves alone, can therefore answer `alt+shift+9` with its own `board open`.
Card 9 installs a hook, process-wide, that writes a `process` task into every hub and beam workspace file a lym process regenerates.
Neither Intent carries a `Bound:` paragraph, while cards 2, 4, 5, 6 and 8 each do.

### [LOW] Closing the listener does not stop kept-alive connections before the lock is released

**Location:** `_lym/plan/05-board-server-lifecycle.md:61`

**Issue:** D11 orders the exit as follows: stop taking new requests, release the lock, wake the waiters, then wait for the requests in flight.
Card 5's step 6 closes the listener and only later calls `http.Server.Shutdown`.
A closed listener refuses new connections, but the server keeps reading requests from established keep-alive connections, such as the open page's.
A page write can therefore start after the lock is released and while a fresh server may be starting.
`boardengine`'s own write lock keeps the board consistent, so the harm is limited to the order D11 states.
`Shutdown` also reports the error from closing the already-closed listener.

### [LOW] The opener's `cmd.Wait` departs from D5's text with no recorded decision

**Location:** `_lym/plan/00-overview.md:34`

**Issue:** D5 says the opener spawn is "reaped with `proc.Reap` as the server's is".
The record's `PATTERN-child-processes` constraint says both spawns are reaped with `proc.Reap`.
The plan's Shared Decisions swap that for one goroutine's `cmd.Wait` feeding the result channel (card 6, `startOpener`).
`pattern/PATTERN-child-processes.md`'s Reap-children rule admits either wait.
`internal/proc/reap.go:14` shows `proc.Reap` discards the exit, which D5's "A failed opener … returns its error" needs.
The substitution is therefore right on the merits, and the PATTERN constraint wins over the record's wording.
What is missing is the record of it: the answer key still names `proc.Reap`, and only the plan carries the change.

### [LOW] The `serve` verb's success answer to a held lock has no reaching test

**Location:** `_lym/plan/06-board-serve-open-verbs.md:114`

**Issue:** D11's acceptance criterion says a second `lym board serve` exits at once with success while the first holds the lock.
Card 5's `TestRun` pins only that `Run` returns `ErrRunning` with nothing written.
Card 6 maps `ErrRunning` to `{"ok":true,"running":true}` and exit 0, but no test runs the verb into that branch.
`TestServeOpenRefusals` stops at the refusals, and `TestBoardServeScenario` runs only `open`.

### [LOW] `TestExpectingRevision` plans two rows for one behaviour

**Location:** `_lym/plan/02-entry-revision-check.md:49`

**Issue:** Row one saves an edit whose revision `Board.View` gave.
Row four saves an edit whose revision `Board.View` gave after a successful write.
`Board.View` reads the saved `board.json` and migrates it unsaved, so both rows exercise the same compare over the same kind of input.
The D8 criterion row four maps to concerns the revision a write returned.
That revision exists only as the server's answer, which card 4's `TestServer` covers with "an edit carrying the `revision` of the entry a successful write answered is saved".

### [LOW] The render and replay file-list comments are left stale

**Location:** `_lym/plan/01-board-view-projection.md:41`, and `_lym/plan/03-viewer-page-asset.md:22`

**Issue:** The file header at `internal/boardengine/render.go:1-2` says Render's outputs are README.md, a project file per project and the root, and a design file per entry with a body.
The comment at `internal/boardengine/replay.go:31` says the replay recognises the rendered overview and the per-entry design and decision files at the Bolt root.
After cards 1 and 3, Render also returns `board-data.js` and `board.html`, and `Claims` claims both.
No card's Intent names either comment.

### [LOW] `docs/overview.md`'s package tree misses `internal/boardserve`

**Location:** `_lym/plan/04-board-server-routes.md:36`

**Issue:** `docs/overview.md` lists every `internal/` package one line each, `internal/boardcli/` and `internal/boardengine/` among them (lines 296-297).
Card 4 creates `internal/boardserve`, and no card adds its line.
Cards 7 and 9 already touch that file, but only its VS Code passages.

### [LOW] Card 6 extends a closed list in `runproc`'s doc

**Location:** `_lym/plan/06-board-serve-open-verbs.md:30`

**Issue:** `internal/runproc/doc.go:8-9` reads "The hub's own processes register under `HubOwner`. Their roles are `RoleSync` and `RoleSlackRelay`."
Card 6 names `RoleBoardServer` among the hub's roles.
That rewrites the sentence, but the plan-writer stencil reserves Extends for adding to a ref without changing what is already there.

### [NIT] The Shared Decisions revision bullet misdescribes promote

**Location:** `_lym/plan/00-overview.md:31`

**Issue:** The bullet says a promote carries its revision straight to the write section.
Card 4 line 65 refuses a task before the write by reading the current `Board.View()`.
That check is needed, because `promoteTo` returns an existing task unchanged rather than refusing it.

## Focus verdicts

- **Opener wait (card 6, Shared Decisions).** `pattern/PATTERN-child-processes.md` licenses `cmd.Wait`, and D5's failed-opener error needs it, but the answer key's text names `proc.Reap`. Filed as F4 so the parent records the change. It is not unlicensed scope.
- **`rerender` Short and `IsRunAgentSession` (card 6).**
  - The export is licensed: D12 makes the verbs refuse "the caller `boardengine`'s run-agent rule refuses", and D13 puts the verbs in `boardcli`, which then needs the rule exported.
  - The Delete-plus-Create pair is the only admissible shape. The hub's `language.yaml` sets no `rename_tools`, and `internal/languageconfig/template.yaml` states that an empty list refuses a symbol `Rename`.
  - The `Short` reword is licensed by D4's last bullet. That bullet names `lym board rerender` as the way forward for a missing `board-data.js`, so a `Short` naming only the README and design docs would mislead the operator it sends there.
  - Not a defect.
- **`BoardServeDirName` (card 5).** `internal/lymdirs/doc.go` says each constant's own doc comment places its directory, and lists by name only the exceptions under `_lym` and the hand-over directory. A hub-level `.lym` constant needs no `doc.go` change. Not a defect.
- **`TestServer` against `TestExpectingRevision` (cards 2 and 4).**
  - `TestServer` needs exactly one stale edit row for the 409-with-`Fresh` mapping.
  - Its create-of-existing-slug row reaches the server's own view pre-check, not the write section.
  - Its stale and removed release rows reach the server's view check.
  - Neither row duplicates card 2.
  - The duplication is inside card 2 (F6).
- **`TestViewerPage` and the route-constant check (cards 3 and 4).** `board.html` is a code asset, not prose. The no-`http://`/`https://` and `board-data.js` script-tag assertions are D2's acceptance criterion, a no-network-load property of the code. The route-constant check pins the Go constants against the paths the static page spells. Neither asserts wording. Not a defect.
- **Reachability (cards 4 and 6).**
  - Every prescribed refusal has a named input and a named row, and `PersistentPreRunE` refuses nothing ahead of `--board-path` (`internal/boardcli/cli.go:125-138`).
  - A stale `release` is refused at the view check, and `TestServer` reaches it there. The write section's re-check is reached generically by card 2's test, since `SetStatusReleasingProject` runs `boardCriticalSection` (`internal/boardengine/projectrelease.go:19-25`).
  - The empty-revision gap is F1. The `serve` verb's success branch is F5.
  - An orch strand name passing the run-agent check is pinned by the existing `internal/boardengine/taskguard_test.go` orch rows over the rule `refuseRunAgent` reuses.
- **Attack surface (cards 7 and 9).** Filed as F2.
- **Callers of `BuildWorkspace` and card order (cards 7, 8 and 9).**
  - A grep finds exactly `internal/fabricengine/hubworkspace.go:178`, `internal/fabricengine/hubworkspace.go:203` and `internal/vscode/workspace_test.go:45`, so card 8's ImpactSummary holds.
  - Card 7 adds only `tasks.json` assertions to `checkPrimeWritesHubWorkspace` and `checkBeamOpensBeamWorkspace`, which hold at its commit. Card 9 adds the workspace-file assertions after card 8's seam lands.
  - Every caller of the deleted `assertAttachTaskOnly` is on card 7. `checkTaskSlugOpensBareFolder` is the caller at `spawn_hub_integration_test.go:273`.
- **D1 to D3 and D8 mapping (cards 1 to 3).**
  - The owner-set write path commits every name `Render` returns (`internal/boardengine/ownerset.go:324-350`), and card 1 keeps `Render` returning `board-data.js` through `renderFiles`. Card 3 adds `board.html` the same way, so D3's owner-set clause is carried with no edit.
  - D8's unsaved migration is carried by `migrateLoaded`, which `boardCriticalSection` and `Board.View` share (card 1), and by `checkExpected` running right after it (card 2). Both have reaching rows.

## Rejected

- None of the fork findings were rejected. The two fixes that change targets (F8, F9) passed `lym loom validate-plan --overlay`, alone and together, with no blocking finding the baseline lacks; F8's passes only with the ImpactSummary its fix includes.
