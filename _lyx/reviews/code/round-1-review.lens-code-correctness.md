```yaml
lens: code-correctness
outcome: reported
```

**Task:** lens `code-correctness`, one concrete case run per function or branch in the changed drift code.

## List: function or branch, case run, verdict

- `LoadConfig` and `decodeStep`:
  - Template `{}` loads as undeclared. Correct.
  - A step with only `dir` set is refused for a missing command. Correct.
  - A command without `{baseline_dir}` is refused. Correct.
  - `timeout_min` 0 is refused. Correct.
  - `verify_max_runs` 0 is refused. Correct.
  - A bad model spec is refused. Correct, but the key checked first depends on map iteration order. Cosmetic.
- `Config.Step` with an undeclared step: refusal names `lyx config drift`. Correct.
- `CountedCapture`:
  - Loom, passed record, `awaited_at` empty: false. Correct.
  - Darn, passed record: true. Correct.
  - Record not found: false. Correct.
- `RunCapture`:
  - Lock held: refusal, no record written. Correct.
  - `Check` fails: the lock is released by `defer` and no record is written. Correct.
  - Command exits 0 but writes no file: failed record with the reason. Correct.
  - Timeout: failed record. Correct.
  - A non-empty baseline directory is moved to the first free numbered backup. Correct.
  - A `runStep` error leaves a failed record, joined with the cause. Correct.
  - A cancelled context gives `exit -1`, recorded as failed. Acceptable.
- `RunSideCapture`: writes to the next numbered directory with no lock or record. Correct.
- **`RunCompare`: INCORRECT, see F1.**
- `SkipList`: scope skip plus accepted, de-duplicated; a withdrawn test is off the accepted list, so it runs again. Correct.
- `runStep`:
  - Slot wait, then the tree kill at timeout, then `release` via the outcome. Correct.
  - On every error path the slot is already released. Correct.
- `ProbeLock`: a missing file reads as not held and creates nothing. Correct.
- `StartCapture`:
  - Lock held: refusal. Correct.
  - Breakaway refused with `FallbackDetach` unset: `ErrBreakawayRefused`. Correct.
  - Breakaway refused with the fallback set: retried as a plain detach. Correct.
  - Child exits before it records: failure naming the child log. Correct.
  - A child refused by `Check` writes no record, so the caller gets only "ended before it recorded its start" and the real refusal is in the child log. Acceptable, because the row pre-checks.
- `CheckMainCapture`:
  - Baseline blocked without `CaptureHaltPrefix`: refused. Correct.
  - Await running: refused. Correct.
  - HEAD off the branch point: refused. Correct.
  - Dirty tree at the Await halt versus at Baseline: different ways forward. Correct.
- `DriftBaseline.drive` and `captureAgain`:
  - No scope, agent records none: Stuck. Correct.
  - Scope off: Done. Correct.
  - `awaited_at` set: Done. Correct.
  - Lock held in loom: Done. Correct.
  - Lock held in darn: waits, then reads. Correct.
  - Passed record: Done. Correct.
  - Running record with a free lock: captures again. Correct.
  - Undeclared capture: Stuck naming `lyx config drift`, without `CaptureHaltPrefix`, so `defer` is not admitted. Matches the decision, which scopes the marker to failed or refused captures.
  - Breakaway refused: foreground capture. Correct.
  - Capture record failed: Stuck with the marker and both ways forward. Correct.
- `DriftAwait.await`:
  - No scope, off, or `awaited_at` set: Done. Correct.
  - No record, running with a free lock, failed, or HEAD moved: Stuck with the right ways forward. Correct.
  - Passed at HEAD: sets `awaited_at` and returns Done. Correct.
- `RecordScope`: all invalid combinations and the second call are refused with no write. Correct.
- `Accept`:
  - Agent session accepts a withdrawn test: refused. Correct.
  - Caller under full, not at a decision halt: refused. Correct.
  - Caller at a non-decision halt under partial: allowed. Correct.
  - Running with the mark closed: refused. Correct.
  - Error or transient state: refused toward resume. Correct.
- `Withdraw` of a test that is not accepted: refused. Correct.
- `Defer`:
  - At the Await halt: sets `awaited_at` and `Deferred`. Correct.
  - At a Baseline capture halt: same. Correct.
  - Capture held: refused. Correct.
- `Waive`:
  - Counted capture exists: refused. Correct.
  - No capture: writes the waiver. Correct.
- `TakeRun`: refuses at max and writes nothing. Correct.
- `DriftVerify.pass`:
  - No scope, off, or waived: Done. Correct.
  - Full with unruled acceptances: decision halt. Correct.
  - No counted capture: halt with the decision prefix, naming `waive`. Correct.
  - Resume from a recorded halt marks the named acceptances ruled. Correct.
  - An error stop leaves the unruled acceptance, and the next entry halts before comparing. Correct.
  - Dirty tree: Stuck. Correct.
  - Compare passes: Done. Correct.
  - Compare fails: agent spawned. Correct.
- `stuckAfterAgent` and `halt`:
  - Under full, every Stuck carries the decision prefix. Correct.
  - Under partial, the final-compare halt gets the prefix only when `Withdrawn` is non-empty. Correct.
- `ScopeRecord` and `Acceptance` doc comments: INCORRECT, see F2.
- `agentActivityFor` and `Active`: reads the capture lock and the compare marker. The marker half is affected by F1.
- `killPairCapture`: the name is computed from `fabricengine.WorktreePath(prime, slug)`. The capture child holds `CaptureJobName(w.root())`, which is `location.WorktreePath()` (`internal/driftcli/wiring.go`, `root`). Equal as long as both yield the same path string. The call is unconditional on Linux and returns false there. No defect found.
- `runproc` role, the `drift-verify` gate key, and the recipe rows: no logic to run. No finding.

## Findings

**F1 — MEDIUM, design — `internal/driftengine/compare.go` (`RunCompare`), `internal/driftengine/spawn.go` (`Active`)**
- `RunCompare` takes the compare marker with a single non-blocking `lock.TryAcquireWriteLock` and refuses with "a compare is already running" when it is not taken.
- `Active` probes the same marker with the same non-blocking take, and the loop's activity reading (`internal/loomcli/looparm.go`, `agentActivityFor`) polls it repeatedly.
- A probe that holds the marker at the instant a compare starts makes that compare refuse with a false "already running".
- The row's own mechanical compare then returns an error, which is an error stop of `Drift-Verify`. An agent's `lyx drift compare` is refused the same way.
- D4 requires the probe never to make a capture refuse and gives the capture a retry window (`LockRetry`, via `AcquireWriteLockWithin`). It says the compare marker is "probed as the capture lock is". The compare has no such retry.
- Fix: take the marker in `RunCompare` with the same bounded-retry acquire the capture uses (`lock.AcquireWriteLockWithin`, a few seconds), and refuse only if the marker is still held after that.

**F2 — LOW, consistency — `internal/driftengine/records.go` (`Acceptance.Ruling`, `RulingUnruled`, `RulingRuled`)**
- The comments say a ruling applies "under partial scope" and describe it as "not yet ruled on" by "a caller".
- The code sets `Ruling` only under `ScopeFull` (`Accept` in `verbs.go`). The decision says the agent session's acceptances stay `unruled` until the parent rules.
- Fix: reword the comments to "under scope full" and "the run's parent".

**F3 — LOW, design — `internal/driftcli/cli.go` (capture child, around line 396)**
- `head` is read through `gitFacts` before the capture lock is taken. `CaptureRequest.Head` stores that earlier value in the capture record, while `Check` re-reads HEAD only for the precondition.
- If HEAD moved between the two reads, the record's `Head` would not match the code that was actually captured. The window is narrow.
- Fix: read HEAD inside `Check`, or after the lock is held, and use that value for `Head`.

## Tool calls
1. Read `internal/driftcli/wiring.go` and grepped `looparm.go` and `driftwiring.go` for the anchor and the commit seam: the manifest lacked `driftcli`'s worktree resolution and the anchor `agentActivityFor` uses.
2. Read `internal/driftcli/cli.go` lines 380–440: the capture child's body (`HoldNamedJob`, `gitFacts`, `Check`) was not in my context.
