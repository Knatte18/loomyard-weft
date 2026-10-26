---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: "internal/pawlengine/beamawait.go:268"
    summary: "A failed re-check whose lifting status write also fails leaves the hold up and the dobby stopped with no notice and a retry up to a minute away"
    fix: "In refuse, when SetStatus(NameBeamAwait) fails, return awaitWake with acted set to true so the wait returns to the one-second floor and the next wake retries the accept soon; keep the request and marker as now."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: "internal/pawlengine/beamawait.go:198"
    summary: "The dobby-down notice is posted on every driver re-entry that finds the dobby down, not once per episode as the Added after Discussion entry says"
    fix: "Record the sent dobby-down notice in the run's await marker, or another file under the beam's .lyx/beam directory, and skip noticeDownDobby while that record stands; clear it when the dobby is seen live or the request is answered. Add a row to TestBeamAwait for a second re-entry with the dobby still down posting nothing."
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: consistency
    location: "internal/orchengine/launch.go:6"
    summary: "Doc comments still describe an orch start the diff removed in StartAction, BeamUpReport.Action and reconcileHub"
    fix: "Reword the comment on StartAction in internal/orchengine/launch.go line 6, on BeamUpReport.Action in internal/orchcli/start.go line 64 and on reconcileHub in internal/orchcli/start.go line 275 to name the bring-up of `lyx beam start`, not an orch start."
    origin: lens:code-affected-set
---

### [LOW] Failed lift of the landing hold is retried at the slow timer

**Location:** internal/pawlengine/beamawait.go:268
**Issue:** In `refuse`, a failed `SetStatus(NameBeamAwait)` returns `awaitWake{}` with `acted` false, so `Call` doubles the wait toward a minute. Until the retry the hold stands, the dobby is stopped and the operator sees no notice. D9 bounds a status left across the hold to the next wake, but that wake can be a minute away.

### [LOW] Dobby-down notice repeats per driver re-entry

**Location:** internal/pawlengine/beamawait.go:198
**Issue:** `enter` calls `noticeDownDobby` whenever the await marker stands, no request stands and the dobby is not live. Each batten re-spawn of the driver re-enters the row and posts the notice again. The Added after Discussion entry asks for one notice, and no sent-notice state is kept.

### [LOW] Stale orch-start wording in doc comments

**Location:** internal/orchengine/launch.go:6
**Issue:** The comment on `StartAction` ("the branch an orch start takes"), the comment on `BeamUpReport.Action` at internal/orchcli/start.go:64 and the `reconcileHub` doc at internal/orchcli/start.go:275 still speak of an orch start, which the diff replaced with `BeamUp` under `lyx beam start`.

## Rejected

- Lens code-correctness F3 (`ensureDobbies` returns a zero entry on an empty `EnsureLive` report) — the fork did not read `EnsureLive` and named no input that yields an empty report for a named worktree.
- Lens code-tests T1 (no test of the load-time drop of a task's `owner_machine`) — `internal/boardengine/store_test.go` carries a case with `owner_machine` on a task, a grouping project and a beam project that wants the key gone for the first two.
- Lens code-tests T2 (adopt refusals for a grouping project) — `internal/adoptcli/adopt_test.go:294-301` has rows for a grouping project below root and below a beam, naming `lyx beam create`.
- Lens code-tests T3 (watcher failed owner read) — `internal/orchengine/watcher_owner_test.go` has the row `FailingReadGoesOnWithNoStop`.
- Lens code-tests T4 (`--anchor` across a Worktree-Create resume) — `internal/battencli/cli.go:46` records the anchor in the seed and `internal/battencli/wire.go:935` passes it to `Topology.Add`; the fork named no missing behaviour.
- Lens code-tests T5 (no quiet notice at Beam-Await) — commit 19 puts the guard in `internal/battencli/wire.go` (`childAtBeamAwait`) with a test in `wire_seams_test.go`; the notice step itself is unchanged.
- Lens code-tests T6 (possible duplicate start refusal rows) — the fork opened no bodies and named no covering test.
- Lens code-affected-set and code-pattern-fit raised nothing else; the declared PATTERN changes land with the declared text and no `Pattern-Departure` trailer appears in the range.
