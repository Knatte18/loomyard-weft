---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/heddlecli/verbs.go:458-466
    summary: renderEveryWorktree skips only prunable entries, so a pair still being made (listed under a worktree staging directory) or a worktree outside the hub is rendered under a wrong hub-level name and reported failed
    fix: In renderEveryWorktree, before taking name, also skip an entry when fabricengine.InWorktreeStaging(entry.Path) is true, and when filepath.Dir of the cleaned entry path is not c.location.HubPath. Keep the existing Prunable skip. Add the staging and out-of-hub skip to the doc comment of renderEveryWorktree and to the "live worktree" wording in doc.go.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/heddlecli/cli_integration_test.go:317-360
    summary: no test pins --no-render on remove or on set and remove with --layer lym, nor the fan-out after an override write; each path wires the flag separately
    fix: Add assertions to the existing OverrideVerbs subtest and WriteRendersEveryLiveWorktree, with no new test function. In OverrideVerbs, assert the set --layer lym and remove --layer lym outputs carry rendered naming the prime, and that each run with --no-render carries no rendered field. In WriteRendersEveryLiveWorktree, add a remove --no-render step that leaves the pair's CLAUDE.local.md and the output without rendered.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/heddlecli/cli_integration_test.go:546-627
    summary: the failed-worktree error branch (NamedWorktree, EventFor or Render failing, with its way forward) and the render_warnings field of rendered worktrees are not covered by any test
    fix: Extend WriteRendersEveryLiveWorktree. Drive the error branch with one hermetic failure of a single worktree, such as the pair's Render failing on a held Bolt lock with c.renderer.BoltLockWait shortened as RenderRefusesWhileTheBoltLockIsHeld does, or a malformed recorded anchor file, and assert failed names that worktree with the "lym heddle render" way forward while the prime still renders. Drive render_warnings by writing a malformed heddle-lym-override.json beside heddle.json, as MalformedOverrideFileIsWarnedOfAndListStillWorks does, and assert render_warnings holds a warning opening with the worktree name.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/heddlecli/verbs.go:470-472
    summary: an Unwritten worktree carries no lym heddle render way forward of its own although doc.go says each failed worktree comes with it, and an error that already names that verb gets the way forward twice
    fix: >-
      In renderEveryWorktree, for the len(report.Unwritten) case append the way forward
      "run `lym heddle render <name>` once the cause is fixed" unless the joined warnings already contain "lym heddle render".
      For the err case append the same way forward only when the error text lacks "lym heddle render",
      as the render verb does at verbs.go:368.
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: internal/heddlecli/doc.go:25-36
    summary: render_warnings, which carries the warnings of rendered worktrees and the whole-listing failure, is printed by set and remove but named in neither doc.go nor the set and remove help text
    fix: In doc.go add a line after the rendered and failed lines saying render_warnings holds the warnings of the rendered worktrees, each opening with its worktree name, and the failure to list the hub's worktrees, in which case no worktree is rendered. In setCmd and removeCmd Long text name render_warnings beside rendered and failed.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/heddlecli/cli_integration_test.go:480-486
    summary: nothing pins that the Bolt sync starts after the renders, the order the commit message and doc.go state; moving syncAfterSave before renderEveryWorktree would pass every test
    fix: In WriteHoldsTheBoltLockAndCommitsNothing, in the spawnSync seam, record whether the prime's CLAUDE.local.md already exists (the set targets claude-file) and assert it did at sync time. No new test function.
  - id: F7
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/heddlecli/cli_integration_test.go:509-511
    summary: the "no worktree file changed" message of the prime status assertion is now misleading, since a set renders the prime and git status only stays equal because the rendered files are excluded
    fix: Change the comment and the failure message at lines 509-511 to say no tracked file of the prime changed, because the rendered files are git-excluded.
---

### [MEDIUM] Staging and out-of-hub worktrees are rendered under a wrong name

**Location:** internal/heddlecli/verbs.go:458-466
**Issue:** `renderEveryWorktree` takes every non-prunable entry `fabricengine.List` returns and names it `filepath.Base(entry.Path)`, then builds the location with `lymcwd.NamedWorktree(c.location.HubPath, name)`.
Fabric lists a pair under its `.fabric-wt-staging-*` directory while it is being made, so the name becomes the pair's final leaf and the location points at a hub path that does not exist yet.
`EventFor` then seeds the exclude list of that missing worktree, fails, and the pair shows up in `failed` under a name it will not keep.
`internal/orchcli/live.go:30-33` skips such entries for the same reason, and `fabricengine/worktreeseam.go:136-138` notes a warp path may lie outside the hub, which this name mapping would also misplace.
The decision record asks for "every live code worktree of the hub", so an in-creation or out-of-hub entry is outside that set.
I read the staging behavior from `orchcli/live.go` and did not run a concurrent creation.

### [MEDIUM] --no-render and the override-layer fan-out are unpinned

**Location:** internal/heddlecli/cli_integration_test.go:317-360
**Issue:** The flag reaches `finishWrite` through four separate call sites (`verbs.go:220`, `:240`, `:303`, `:305`).
Only `set` of a repository entry is tested with `--no-render`.
`remove --no-render` and both `--layer lym` paths, and the fan-out after an override write, have no assertion, so a dropped `!noRender` argument on any of them passes every test.

### [MEDIUM] The failed-worktree error branch and render_warnings are untested

**Location:** internal/heddlecli/cli_integration_test.go:546-627
**Issue:** The new subtest reaches `failed` only through the `Unwritten` case (an unmarked file).
The `err != nil` case of the switch (`NamedWorktree`, `EventFor` or `Render` failing), with its way forward, and the `default` case that fills `render_warnings`, are added behavior no test reaches.

### [LOW] Failed worktrees do not consistently carry the render way forward

**Location:** internal/heddlecli/verbs.go:470-472
**Issue:** `doc.go` says each failed worktree comes with its cause and the `lym heddle render <worktree>` way forward.
The `Unwritten` case builds its text from the warnings alone, so it names the verb only when a warning happens to.
The error case appends the way forward unconditionally, so a `Render` error that already names `lym heddle render`, such as the Bolt lock wait running out, prints two ways forward; the `render` verb guards this with a `strings.Contains` check.

### [LOW] render_warnings is undocumented

**Location:** internal/heddlecli/doc.go:25-36
**Issue:** `finishWrite` prints `render_warnings` for the warnings of rendered worktrees, and `renderEveryWorktree` returns the listing failure there too, with `rendered` and `failed` both empty.
Neither `doc.go` nor the `set` and `remove` help names the field, so a caller who reads only the documented `rendered` and `failed` misses a failure to list the hub's worktrees.

### [LOW] The sync-after-render order is unpinned

**Location:** internal/heddlecli/cli_integration_test.go:480-486
**Issue:** The commit message and `doc.go` state that the sync starts after the renders so none waits behind its lock.
`WriteHoldsTheBoltLockAndCommitsNothing` only checks the lock is free at sync time, which holds in either order.

### [NIT] Misleading prime status message

**Location:** internal/heddlecli/cli_integration_test.go:509-511
**Issue:** The assertion message says "no worktree file changed", but `set` now renders into the prime; the status stays equal only because the rendered files are in the exclude list.

## Rejected

- Lens code-correctness finding 2 (listing failure leaves `failed` empty): the failure is reported with its cause and way forward in `render_warnings`, and the verb stays ok because the change is saved, as the entry's report-the-worktrees ask allows; the remaining gap is the missing documentation, kept as F5.
- Lens code-correctness finding 4 (renders wait behind a detached sync's lock hold, each up to the default wait): the lens did not verify how long the sync holds the lock, a render waits only until the lock is released, and the same wait applies to any render; the commit orders the sync last to avoid its own sync, which holds.
- Lens code-tests items 15 and 19 (fan-out at a non-`.` anchor, a refused change leaving worktree files untouched): the fan-out shares `renderInto` and `NamedWorktree` with the `render` verb, whose anchor is pinned by `RenderWritesFileTargetsAtTheAnchor`, and a refusal returns from `finishWrite` before any render, which `RefusalsSaveAndSyncNothing` covers by asserting no save.
- Lens code-tests items 10, 14 and 20: the `nonNil` path, an empty-warning cause and "ChangeStore renders no worktree" add no behavior the pinned tests would miss, and the last is the absence of code.
- Lens code-affected-set finding 1 is merged into F1 and raised there at MEDIUM; its LOW severity and `scope` class yielded to the correctness lens's evidence of a wrong name and a spurious failure.
