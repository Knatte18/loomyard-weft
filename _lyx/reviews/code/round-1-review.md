---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/runlevel.go:1180
    summary: deadEndBatchError refuses a dead recovery at the cap that holds an unrendered amendment, though recover-batch could still spawn for it
    fix: In deadEndBatchError, in the `case bs.Status == DigestStatusDead` arm, leave `why` empty when `rendersNewAmendment(bs)` is true, so the dead-with-no-retry refusal matches the exhausted arm's amendment exemption; add a non-refusing row for it to TestRun_RefusesBeforeSpawn's sibling table (see F3).
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/runlevel.go:1172
    summary: A failed git read inside RecoveryRetry is read as "no retry", so a transient git fault at run entry refuses the run as a dead end with a misleading reason
    fix: In deadEndBatchError, do not treat RecoveryRetry's false as a dead end when the false came from a failed git read; either have the dead-end check use a variant of the retry rule that returns the git error and pass it up as a plain run-entry error naming a re-run, or limit the dead arm to bs.RecoveryStartSHA == "" or a successful committedSince(false) result.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/websterengine/runlevel_test.go (TestRun_RefusesBeforeSpawn rows)
    summary: The dead-end refusal has no negative rows for the states it must not fire on
    fix: Add rows, in one table-driven test and not one test each, where Run reaches Master without ErrBatchDeadEnd for a dead recovery with RecoveryRetry true, a terminal batch at the cap with an unrendered amendment, and a done batch.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/websterengine/recordbatch_test.go:1388
    summary: TestRecordBatch_NeedsParentWaits was inserted between the doc comment of TestRecordBatch_MalformedReportYAMLErrors and its function, leaving the latter undocumented
    fix: Move the two comment lines beginning "TestRecordBatch_MalformedReportYAMLErrors proves" back to sit directly above `func TestRecordBatch_MalformedReportYAMLErrors` (line 1429), leaving only the NeedsParentWaits comment above its own function.
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/runlevel.go:1019
    summary: The outcome-done check reports a waiting batch as begun and not recorded terminal, hiding the cause
    fix: "In the offender switch, add a `case waitsOnParent(bs):` before `case !bs.Terminal:` that appends NN-slug followed by a note that the batch waits on its parent with waitingNeed(bs)."
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-pattern-fit
    location: contracts/stencils/webster/webster-body-implementer.md, the line that ends with the need clause and continues into the head_sha sentence
    summary: Two sentences share one line on a line the diff edited
    fix: Break the line after "which `need` names." so `head_sha` starts its own line.
---

### [LOW] Dead-end check refuses a batch a recovery could still take

**Location:** internal/websterengine/runlevel.go:1180
**Issue:** `RecoveryRetry` returns false whenever recoveries reach the cap, whatever the amendment, while `RecoverSpawnOrAttach` still allows a spawn at the cap when an unrendered amendment exists.
So a dead recovery at the cap with such an amendment is refused by `Run` although the decision's bound ("no unrendered amendment") says it should proceed.

### [LOW] Transient git fault read as a dead end

**Location:** internal/websterengine/runlevel.go:1172
**Issue:** `RecoveryRetry` folds a failed git read into false (logged at Warn).
`deadEndBatchError` discards that distinction, so a transient fault blocks the run with a text claiming the recovery earns no retry.

### [LOW] Missing negative rows for the dead-end refusal

**Location:** internal/websterengine/runlevel_test.go
**Issue:** Only the two refusing states are tested; a widening of `deadEndBatchError` would pass every test.

### [LOW] Test inserted inside another test's doc comment

**Location:** internal/websterengine/recordbatch_test.go:1388
**Issue:** The two comments run together and `TestRecordBatch_MalformedReportYAMLErrors` lost its doc comment.

### [NIT] Waiting batch misreported at run exit

**Location:** internal/websterengine/runlevel.go:1019
**Issue:** A waiting batch surfaces as "begun, not recorded terminal".

### [NIT] Two sentences on one stencil line

**Location:** contracts/stencils/webster/webster-body-implementer.md
**Issue:** CLAUDE.md requires one sentence per line in every `.md` file.

## Rejected

- Missing friction note on the `ErrBatchDeadEnd` refusal (pattern-fit F1): no Go code in the webster run-entry paths writes a friction note, including the existing pending-audit refusal, so the new refusal matches the established behavior.
- `internal/fabriccli/roleguard.go:59` still says "report status: FAILED" (affected-set F1): the decision record puts the generic FAILED-and-the-orch-runs-it clause out of scope, and the implementer body maps that clause to `NEEDS_PARENT` for a webster implementer.
