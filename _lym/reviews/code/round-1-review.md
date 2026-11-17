---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/beamcli/hubstop.go:113
    summary: The detached stop starts in the prime's worktree root, so in a hub whose prime has a subpath anchor its strict cwd resolve refuses and nothing is stopped while the caller reports a pid
    fix: In spawnDetachedStop set cmd.Dir to prime.AnchorPath() instead of prime.WorktreePath(), as orchcli/spawn.go and fronttask/hub.go do for their lym children, and update the log fields to match.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/orchcli/stop.go:41
    summary: BeamDown's doc and orchcli's package doc claim a caller inside the orch's session has finished every other step before its process ends, but stopOrch's strand removal reaps the pane's descendants, the caller included, before the dobby or the orch process is ended
    fix: In internal/orchcli/stop.go's BeamDown doc and internal/orchcli/doc.go, drop the claim that a caller inside that session has finished every other step; state instead that the strand removal ends the orch's pane and every process below it, so a caller inside that pane ends there and the later steps run only for a caller outside it. Leave the step order as it is.
    origin: assist:assist
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/beamcli/hubstop_test.go:420
    summary: The stop --all command paths and the worktree selection have no test - the slug-beside-all refusal, the caller's spawn and its --detached argument, and hubStopWorktrees' split of beams from task pairs
    fix: Add a stop b --all row asserting the exact refusal message to the refusal table in internal/beamcli/start_integration_test.go. Factor spawnDetachedStop's argument and directory building into a function the test reads, and assert it carries --all and --detached and the prime's anchor directory. Add a test of hubStopWorktrees over a hubforge hub with a prime, one beam and one task pair, asserting the beam lands in beams and the task pair alone in tasks.
    origin: lens:code-tests
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/beamcli/hubstop_test.go:437
    summary: TestRunHubStop exercises only a failed session down; failed pause, worktree list, beamDown, reap and lift, and the ConfirmationLifted mapping, are unpinned
    fix: Extend TestRunHubStop's table with a per-step error field and rows where pause, worktrees, beamDown, reap and lift each fail, plus a ConfirmationLifted row. For each, assert the full call order still runs, the exact failure strings, that a failed lift leaves PauseLifted false, that a failed beamDown's report is still appended, and for the existing session-down row that the failure names t1 and t1 is absent from report.Pairs.
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/orchcli/stop.go:63
    summary: beamDown returns before its last step when the dobby read or end fails, so the orch's registered process is left running though the hub stop's criterion says it leaves none
    fix: In beamDown, on a dobbyLive or endDobby error, still call c.endOrch and return errors.Join of both errors with the report's OrchProcessEnded set from the end; keep stopOrch's start.lock refusal returning early. Add a beamDown row where endDobby fails and assert end-orch still runs.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: design
    location: internal/beamcli/hubstop.go:318
    summary: A beam whose _lym junction or origin record HubOrchs cannot read is put in tasks, so it is downed as a task pair and never gets BeamDown
    fix: In hubStopWorktrees, read each non-main worktree's origin through fabricengine.ReadOrigin and put a worktree in tasks only when the origin reads and lacks the beam mark; log and skip one whose junction or origin cannot be read, and return its name so runHubStop records it in Failures.
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/orchcli/beamdown_test.go:914
    summary: No row makes endOrch fail, so beamDown's wrapped error and the report it returns with it are unpinned
    fix: Add a row to TestBeamDown_StopsOrchThenTheDobbyThenTheOrchProcess whose endOrch fake returns an error, asserting the error names the worktree's orch process and the report keeps OrchRemoved and DobbyEnded.
    origin: lens:code-tests
  - id: F8
    severity: NIT
    class: design
    location: internal/beamcli/hubstop.go:384
    summary: After a ConfirmationLifted outcome the no-op Lift succeeds and the report says pause_lifted true for an entry the operator lifted
    fix: In runHubStop, call steps.lift only when paused.Owned is set and paused.Outcome is not ConfirmationLifted.
    origin: lens:code-correctness
  - id: F9
    severity: NIT
    class: consistency
    location: docs/overview.md:385
    summary: The internal/hibernatecli tree line does not name the PauseHub entry the hub stop calls
    fix: Append to the internal/hibernatecli line in docs/overview.md that it also holds the hub pause and confirmation lym beam stop --all runs.
    origin: lens:code-affected-set
---

### [BLOCKING] The detached stop starts outside the prime's anchor

**Location:** internal/beamcli/hubstop.go:113

**Issue:** `spawnDetachedStop` sets `cmd.Dir = prime.WorktreePath()`.
The child re-enters `stopAll`, whose `actingLocation` calls `lymcwd.Resolve`, which requires the cwd to equal the anchored directory exactly (`internal/lymcwd/lymcwd.go`, Resolve).
In a hub whose prime records a subpath anchor, the detached child refuses before the pause, and its refusal goes nowhere, since its stdout is nil.
The caller has already printed the pid as success, so the button reports a stop that never ran.
The other lym child spawns in the tree work at `AnchorPath()` (`internal/orchcli/spawn.go:34`, `internal/fronttask/hub.go:156`).

### [MEDIUM] The in-session claim of BeamDown's docs is false

**Location:** internal/orchcli/stop.go:41

**Issue:** The new doc text says a caller inside the orch's session has finished every other step before the orch process ends.
`beamDown` removes the strand first, and reed's pane removal reaps the pane's whole descendant closure, force-killing stragglers, with no sparing of the caller (`internal/reedengine/lifecycle.go`, reapPaneChildren).
A `lym beam stop` run inside the orch's pane therefore dies at the first step, before the dobby is ended.
That order predates this diff, and the hub stop runs detached, so the new comments, not the code, are what the diff gets wrong.

### [MEDIUM] stop --all's command paths and worktree selection are untested

**Location:** internal/beamcli/hubstop_test.go:420

**Issue:** `TestRunHubStop` calls the private sequence over fakes, `worktrees` included.
Nothing reaches the slug-beside-`--all` refusal, the spawn's argument list, whose missing `--detached` would make the child spawn itself again, or `hubStopWorktrees`, which decides which worktrees are downed as task pairs.
The refusal table in `start_integration_test.go` has a `start` row for the same exclusion and none for `stop`.

### [MEDIUM] Only one of the sequence's failure paths is pinned

**Location:** internal/beamcli/hubstop_test.go:437

**Issue:** The rubric-level promise that the stop goes past every failed step is pinned only for a failed session down.
The fakes for pause, worktrees, beamDown, reap and lift never fail, and no row returns `ConfirmationLifted`.
The one failure row asserts only the count of failures.

### [LOW] An earlier failure skips the orch process end

**Location:** internal/orchcli/stop.go:63

**Issue:** A failed dobby read or end returns before `c.endOrch()`.
In the hub stop the failure is named, but the orch's Claude process stays up, against the acceptance criterion that the stop leaves none.

### [LOW] An unreadable beam is downed as a task pair

**Location:** internal/beamcli/hubstop.go:318

**Issue:** `HubOrchs` skips with a warning a pair whose junction or origin cannot be read (`internal/orchcli/live.go`).
`hubStopWorktrees` classes every worktree not in that list as a task, so such a beam goes through reed's pair path and never through `BeamDown`.

### [LOW] beamDown's endOrch error path has no row

**Location:** internal/orchcli/beamdown_test.go:914

**Issue:** The `endOrch` fake never fails, so the new wrapped error is unpinned.

### [NIT] pause_lifted reads true after an operator's resume

**Location:** internal/beamcli/hubstop.go:384

**Issue:** `PauseAndConfirm` sets `Owned` before a grace that the operator's resume ends, and `Lift` of a gone entry succeeds, so the report claims a lift the stop never made.

### [NIT] The hibernatecli tree line misses PauseHub

**Location:** docs/overview.md:385

**Issue:** The line lists the hibernate commands, wiring, carrier and spawn, and not the hub stop's pause entry the diff adds.

## Rejected

- Assistant's EndSubject-versus-EndRole scope finding: the measuring stick is the amended `## Goal`, which asks for the end "through runproc"; D9's naming of `EndSubject` is the record writer's own section, and `EndSubject` over the prime's hub-owner record would end other hub-owned processes too.
- Assistant's BLOCKING redesign of the in-session path: the remove-then-end order predates the diff and the hub stop runs detached; the false doc claim is kept as F2.
- code-affected-set's LOW on the `beamcli` row of `pattern/PATTERN-cli-cobra.md`: the row already left out `orchcli`, `adoptcli` and `dobbyengine` before this diff, and the directive declares no PATTERN change for a fixer to land.
- code-affected-set's note that the summary overclaims the buttons test: the test reads the running GOOS's template, so the two templates are each pinned on their platform.
- Handler's check of `carry`'s withdraw return, the prior request in place of a zero one: the carrier reads it only for a log line's phase.
