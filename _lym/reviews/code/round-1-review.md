---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/verbs.go:465
    summary: A chain of scope replacements loses the skip list the capture was taken under, so a capture lacking a skipped test's profile is counted under a later full scope.
    fix: In RecordScope, stop inferring what the capture skipped from the current record's Skip list, since a replacement overwrites that list. Either store the skip list in the capture record when the capture is taken and compare against it, or drop the capture on any replacement that leaves scope off for a scope other than off. Add a TestRecordScope row with a capture taken under partial skipping a, a replacement to off, then a replacement to full, wanting the capture removed.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/driftwiring.go:141
    summary: Nothing pins that the production wiring hands the engine the anchored directory rather than the worktree root, which is the fix for issue 23 itself.
    fix: Add a test with a subdirectory anchor, where the fixture's AnchorRel is not ".", that wires internal/driftcli (startMainCapture, the compare and side-capture commands, the capture child) and internal/loomcli driftDeps, and asserts the spawned child's dir and the command's cwd equal location.AnchorPath(). Reverting any of those call sites to WorktreePath() must fail it.
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/driftengine/verbs.go:469
    summary: When os.Remove of the capture record fails after the scope was replaced, the new scope is stored and a retry no longer sees the skipped test difference, so the stale capture stays counted.
    fix: In RecordScope, remove the capture record before UpdateScope writes the replacement, or remove it inside the same locked update, so a failed removal leaves the old scope in place and the call can be retried.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/driftengine/steprun.go:149
    summary: acquireSlot now records the anchored directory as gateslot Holder.Worktree, which is documented as the worktree and is the worktree root for every other holder.
    fix: Keep the worktree root for the holder identity, for example by carrying a separate worktree root field on stepRun and the capture and compare requests, or change the Holder.Worktree doc comment in internal/gateslot/pool.go to say it may be a path inside the worktree. Add an assertion on the recorded holder in a steprun test.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/baseline.go:147
    summary: Every StartCapture error other than a refused breakaway becomes a capture halt offering lym drift defer, including a failed binary lookup, a child log that cannot be created and a held capture lock.
    fix: In captureAgain, turn only the child-start failures from startFailure into the capture halt, and keep other StartCapture errors returned as before, so defer is not offered for an environment fault. A held capture lock keeps its own refusal wording.
---

### [MEDIUM] Scope replacement chain loses the skip list of the capture

**Location:** internal/driftengine/verbs.go:465
**Issue:** `dropCapture` compares the new skip list with `cur.Skip`, the current record's list. A capture taken under `partial` skipping `a` survives a replacement to `off`, which sets `cur.Skip` to nil. A later replacement to `full` then finds an empty `cur.Skip`, keeps the capture, and compares test `a` against a baseline that holds no profile of it.

### [MEDIUM] The anchored start directory is not pinned at the wiring

**Location:** internal/loomcli/driftwiring.go:141
**Issue:** The engine tests build the anchor and the worktree root as the same temporary directory, and the driftcli and loomcli wiring fixtures use `Anchor: "."`. The change that fixes issue 23 sits in `w.anchor()` and `location.AnchorPath()` at the call sites, and a revert of those fails no test. `TestStartCapture` pins only that the engine uses the `AnchorDir` it is told.

### [LOW] A failed capture removal leaves the scope replaced and the capture stale

**Location:** internal/driftengine/verbs.go:469
**Issue:** The scope is written first and the capture removed after. When the removal fails with an error other than not-exist, the error is returned but the replacement stands. A retry computes `dropCapture` against the already replaced `cur.Skip`, finds no difference, and never removes the capture.

### [LOW] Gate-slot holder identity became the anchored directory

**Location:** internal/driftengine/steprun.go:149
**Issue:** `gateslot.Holder.Worktree` is documented as the worktree the holding run works in, and `gatecli/run.go:584` passes `location.WorktreePath()`. In an anchored hub the drift holder now shows a subdirectory where every other holder shows the worktree root. The readers found only print or log the value, so nothing breaks, but the diff changed it with no stated need.

### [LOW] Every non-breakaway start error is offered a defer

**Location:** internal/driftengine/baseline.go:147
**Issue:** The new halt covers any `StartCapture` error that is not `ErrBreakawayRefused`. A failed binary lookup or an unwritable child log is an environment fault, and the halt's `lym drift defer` way forward skips regression for it. The capture-lock-held refusal also becomes this halt, while `Defer` itself refuses at a held capture.

## Rejected

- Lens code-tests, replacement keeps the other record fields with no assertion on `Deferred` or `WaivedReason`: a replacement is refused once `AwaitedAt` is set, and `WaivedReason` is written only at Drift-Verify, so neither field can be set at a replacement the verb admits.
- Lens code-affected-set, `gateslot.WaitDir` already given `AnchorPath()`: unchanged by the diff and not a defect.
- Lens code-correctness, `drift.enabled` switched off while a capture child runs: the pass returns Done and the child's record is ephemeral; the lens itself judged it edge-only.
