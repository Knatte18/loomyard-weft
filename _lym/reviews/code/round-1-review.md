---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/loomcli/sharedbootstrap.go:301
    summary: "Status window is titled <handle>: <recipe> where the amended entry asks for <handle>: overview; the departure is recorded only in the change description, with no Added after Discussion entry and no Pattern-Departure trailer"
    fix: "Leave the code in place and defer to the run's parent: the parent records the <handle>: <recipe> title with lym loom decision add, or refuses it and orders <handle>: overview. This is outside the fixer's write surface."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/wire.go:840
    summary: "commandDeps Title, the Open-Window titled name <handle>: <slug>, has no test of its own; a wrong slug, a wrong board dir or a nil Title would pass every test, and a nil Title panics the default Open-Window row"
    fix: "Add a case in internal/battencli that builds commandDeps over a hub whose board holds the slug with handle T4 (boardkit.WriteBoard under fabricengine.BoardDir(hub)) and asserts Title() is T4: <slug>, and the bare slug when the hub has no board."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/sharedbootstrap.go:294
    summary: "A live current-build status or runs strand is kept without comparing its title, so a handle assigned or moved after the window opened (promote, type change) never reaches the title until the strand is replaced"
    fix: "Either accept it and state in internal/loomcli/doc.go and internal/orchcli/cli.go that the title is fixed when the strand is added or replaced, or make a title mismatch a reason to replace in ensureStatusStrand and ensureRunsStrand (needs the live window's title from the strand status)."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/sharedbootstrap.go:301
    summary: "Nothing pins that ensureStatusStrand passes statusWindowTitle into the AddSpec; reverting to c.recipe passes every test"
    fix: "Where the ensureStatusStrand tests have a fake reed capturing the AddSpec, add a row asserting Display.Title is the handle-opened title for a board that names the run's handle."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/runs.go:80
    summary: "Nothing pins that ensureRunsStrand puts c.runsWindowTitle() into the spec it adds or replaces with; the wired-title test calls the func directly and start_test.go stubs it"
    fix: "In the start harness make the runsWindowTitle func return a distinctive value and assert the AddSpec passed to AddStrand or ReplaceStrand carries it as Display.Title."
---

### [MEDIUM] Status window title departs from the entry without a record

**Location:** internal/loomcli/sharedbootstrap.go:301
**Issue:** The amended entry asks for `<handle>: overview` for the loom status window. The diff titles it `<handle>: <recipe>`. The decision record has no `Added after Discussion` entry, and the commit carries no `Pattern-Departure:` trailer. Only `_lym/landing/summary.md` explains it, as a writer's claim. The reason is sound: `7c23591e` already replaced the bare `overview` default with the recipe name, so the entry's premise is stale. It is still an unrecorded parent correction, and a revert is the wrong fix.

### [MEDIUM] Open-Window titled name has no test

**Location:** internal/battencli/wire.go:840
**Issue:** The entry asks tests to pin the titled and fallback names of both windows. `TitleWithHandle` is tested alone, and `TestCommandProducer` feeds a stub `Title`. The wiring that builds `<handle>: <slug>` from the hub's board and the task slug is covered by no test.

### [LOW] A kept strand keeps its old title

**Location:** internal/loomcli/sharedbootstrap.go:294
**Issue:** `resolveStatusStrandAction` and the runs-strand branch keep a strand that is live with a matching GUID and build, and compare no title. A handle that appears or moves after the window opened is not shown until the strand is replaced. The entry says the handle is read when the window opens, so this is defensible, but the docs say the window is opened with the handle without saying it is fixed from then on.

### [LOW] Status strand title wiring is unpinned

**Location:** internal/loomcli/sharedbootstrap.go:301
**Issue:** `TestStatusStrandAddSpec` and `TestStatusWindowTitle` each call their helper directly. The call in `ensureStatusStrand` that joins them is untested.

### [LOW] Runs strand title wiring is unpinned

**Location:** internal/orchcli/runs.go:80
**Issue:** `TestRunsWindowTitle_WiredFromTheWorktreeTheVerbRunsIn` calls `runsWindowTitle()` directly, and the start harness stubs it. Nothing shows the title reaches the spec `ensureRunsStrand` hands to reed.

## Rejected

- Title laziness untested (lens:code-tests, `internal/shedadapters/command.go:120`): an eager call would only cost a board read and change no output, so a row for it would pin cost and not behavior.
- Warn log on an unreadable board untested (lens:code-tests, item 3): log text is not behavior a test needs to pin, and the lens raised no finding.
- Slug lookup per worktree kind: `location.WorktreeName` is the worktree directory, which equals the entry slug in a task worktree and a beam worktree, and the prime has no entry so it falls back; batten passes the task slug. No defect.
- Missing board file: `Store.Load` returns an empty list with no error, so no Warn fires and nothing is written; this matches the fallback `TestTitleWithHandle` pins.
- `{title}` placeholder: filled only when present and quoted like the other values, so a hub-layer command holding `{slug}` keeps working; the placeholder lists in `template.yaml`, both `doc.go` files and `docs/overview.md` match the code.
- Scope: `lym reed window` with no `--title` and the status-bar `term` button are untouched, and the `runsWindowTitle` field change has no other reader.
