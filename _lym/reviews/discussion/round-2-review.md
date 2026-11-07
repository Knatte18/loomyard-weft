---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D2 last bullet, D3 fold, D7 internal/fabricengine anchor and origin (endedHub)
    summary: D2's board HEAD equals upstream tip check reads as the tracking ref, so a builder commit made straight into the records bare, as endedHub's originBoard makes, passes the build and D3's per-copy push --force drops it
    fix: In D2's last bullet, define the upstream tip as the records bare's own branch tip, read from the bare, not the board's tracking ref, and say the build fails naming the key when the board's HEAD differs from it. In D7's list of steps that stay in the test after the copy, add a step that commits straight into a bare, naming endedHub's originBoard. Add a D2 acceptance criterion - a build that commits into the records bare after the board's push fails naming the key.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D1, lines 33-35 and 47-50
    summary: D1 never says how the reader reaches the common dir from the directory a List caller passes - callers pass AnchorPath(), a subdirectory at anchor backend, and pair paths that may be gone and whose failure triggers the caller's fallback
    fix: In D1, state that the reader takes any directory inside a worktree and finds its repository by walking up as git's discovery does, and that a directory that does not exist is an error with no walk-up, so the List-on-a-gone-pair fallbacks in internal/fabricengine/junction.go and internal/fabricengine/scratchlink.go still fire. Add D1 acceptance criteria - List from a subdirectory of a worktree equals List from its root, and List on a missing directory returns an error.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D2 Bound, lines 95-99, and the D7 review criterion
    summary: D2's Bound lists only process-wide calls the build itself makes, but a build runs under the package-global test seams its first caller installed and may itself make once-only registrations such as boardkit.InstallReplay, and either result reaches every later copy
    fix: In D2's Bound, add that a build runs under whatever process-global seams and registrations are installed when its first caller reaches it, and state the rule - a build calls no package-global seam setter or once-only registration, and a test that installs a process-global seam takes its copies before installing it. Extend the D7 review criterion to check both.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 Bound, line 114, and the first D3 acceptance criterion
    summary: D3 says a copy's board stays clean against its HEAD and upstream, but loomcli beamHub, which D7 moves, writes board.json and the shortname file into the board uncommitted, so its copies start dirty
    fix: In D3's Bound, replace the clean-board sentence with - a copy's board keeps the working tree its build left, with HEAD equal to its upstream, its history as built and the binding folded into the tip. Limit the clean-board part of the first D3 acceptance criterion to a shape or a registration whose build commits what it writes.
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md Scope Out, D10 and Open risks
    summary: The record calls gate-tier-binaries-only in flight, but it landed on main as 3aca2e50 with {test_targets} opt-in, and the hub's gate.yaml tiers still run {targets}
    fix: In Scope Out, D10 and the first Open risks line, replace in flight with landed on main as 3aca2e50, with {test_targets} opt-in and the hub's tiers not yet switched; keep D10's two expectations, the 48 s one holding once the hub's tier commands carry {test_targets}.
---

### [MEDIUM] D2's upstream check cannot see a commit made straight into the records bare

**Location:** `_lym/discussion/decision-record.md` D2 last bullet, D3's fold, D7's `endedHub` move.

**Issue:** `endedHub` runs `publishBoard`, which commits and pushes, then `originBoard`, which commits the done board straight into `hub.RecordsBare` (`internal/fabricengine/originreconcileended_integration_test.go:58-71`, `:87-100`).
The board's HEAD still equals its tracking ref, so a check against `@{u}` passes, and D3's per-copy `push --force origin HEAD` (`internal/hubforge/template.go:187`) rewinds the copy's bare and drops origin's done commit.
D3's own criterion, the bare's branch tip is the board's HEAD, then holds only because the commit was lost.
Three lenses found it; interactions rated it BLOCKING.
It is MEDIUM here because the dropped commit fails `TestReconcileOrigin_Ended`'s rows loudly rather than passing them, and the fix is one rule plus one named step.

### [MEDIUM] D1 does not say how the reader finds the repository from a caller's directory

**Location:** `_lym/discussion/decision-record.md` D1.

**Issue:** `List` runs `git worktree list` at `sourceDir` (`internal/fabricengine/worktreelist.go:31`), and git accepts any directory inside a worktree.
Callers pass `l.AnchorPath()`, a subdirectory at anchor `backend` (`worktreelist.go:114`, `:141`, `internal/hubgeom/parentof.go:36`), where `dotgit.Read` finds no `.git` (`internal/dotgit/dotgit.go:32-36`).
`junction.go:624-626` and `scratchlink.go:175-177` call `List` on a pair path that may be gone and fall back when it fails; a plain walk-up such as `dotgit.FindRoot` climbs past a missing directory (`dotgit.go:80-108`), so that failure would no longer come.
The parity rules and criteria cover neither case, and the record says "List keeps its callers".
Bound-breaker and existing-paths raised the subdirectory half at LOW.

### [MEDIUM] A build inherits its first caller's process-global seams

**Location:** `_lym/discussion/decision-record.md` D2 Bound.

**Issue:** The build runs in whichever test copies the template first.
Package-level seams that `Add` and the build path read, such as `SetAddFailAfterWarpHookForTest` and `SetWeftHeadSHAForTest` (`internal/fabricengine/export_test.go:53`, `:631`, `:641`) and `hubreconcile.SetSpawnBoardSyncForTest` (`internal/hubreconcile/ensure_integration_test.go:477`, with `newStaleHub` taken after the swap at `:506`), are live during that build, and their effect lands in every later copy.
A builder can also make a once-only registration such as `boardkit.InstallReplay` (`internal/testkit/boardkit/boardkit.go:20-31`), which is never undone and changes later tests that did not ask for it.
D2's Bound and D7's review criterion name only `os.Setenv`, `os.MkdirTemp`, `git config --global` and `Context()`.
Bound-breaker's once-registration finding is merged here.

### [LOW] D3's clean-board claim does not hold for loomcli's beamHub

**Location:** `_lym/discussion/decision-record.md` D3 Bound and its first acceptance criterion.

**Issue:** loomcli `beamHub` writes `board.json` and the shortname file into `hub.BoardDir()` without committing (`internal/loomcli/projectrecord_integration_test.go:29-46`).
D2 checks only HEAD against upstream, and the fold amends only `.lym-warp`, so every copy of that template starts with a dirty board, as today's fixture does.
The claim is right for shapes and wrong for that registration.
Interactions, walkthrough and bound-breaker each raised it.

### [LOW] gate-tier-binaries-only has landed

**Location:** `_lym/discussion/decision-record.md` Scope Out, D10, Open risks.

**Issue:** `3aca2e50` on `main`, "Gate: {test_targets} starts only the test binaries of the tier", landed after the record was written; `lym board get` returns no `gate-tier-binaries-only` task.
Its commit message says `{test_targets}` is opt-in, and `_bolt/_lym/config/gate.yaml` still runs `{targets}` in every tier.
D10's two expectations still fit; only the in-flight wording is stale.
The writer can settle it by citing the commit, so it is `consistency`, not `decision`.
