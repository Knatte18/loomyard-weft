---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/15-drift-judge-seat.md:65
    summary: The judge snapshot counts loom's own weft commits of the tracked run status as judge edits, so a resumed Drift-Verify with an open judge mark, or one resumed after a judge-change Stuck, is Stuck again on every resume.
    fix: >-
      Card 2: add the general read-only seam `plan:internal/fabricengine#WeftPathsChangedSince` -> `func WeftPathsChangedSince(l *lymcwd.Location, rev string) ([]string, error)`,
      the anchor-relative weft paths whose content differs between rev and the weft HEAD, with a TestCheckoutStates row for it.
      Card 15: list it under Uses and add a `Deps` seam over it.
      `Changes` takes the paths loom itself writes on the weft during a row, the run's shed directory (`shedrun.RunDir`'s anchor-relative path, holding status.json) and `judgement.yaml`, as exempt.
      A moved weft HEAD counts as a change only when `WeftPathsChangedSince(snapshot head)` names a path outside the exempt ones, and the weft's Changed and Untracked lists drop the exempt paths before the compare.
      TestJudgeSnapshot gains a row where a commit changing only the run's status.json after the snapshot is no change.
      Card 16: TestDriftVerify gains a row where an open judge mark is resumed after loom committed the row's status and the judge is resumed, not stopped.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/25-driftengine-retire-acceptance.md:48
    summary: Card 25 deletes verifyFixture.agentAccept and verifyFixture.sessionAccept, which card 16 already deletes, so delete-target-gone fires at Webster's dispatch.
    fix: >-
      Card 16: drop `internal/driftengine#verifyFixture.captureForAcceptance`, `internal/driftengine#verifyFixture.agentAccept` and `internal/driftengine#verifyFixture.sessionAccept` from its Delete list, leaving the three helpers in place untouched by its fixture rewrite.
      Card 25: add `internal/driftengine#verifyFixture.captureForAcceptance` to its Delete list beside the two it already deletes.
      Dropping the two from card 25 alone is not admissible: its statusAt re-sign then fails caller-uncovered on verify_test.go:227 and 244, and dropping only card 16's two fails caller-uncovered on captureForAcceptance.
      The validate-plan overlay passes with this shape.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:affected-set
    location: _lym/plan/04-drift-config-keys.md:3
    summary: Cards 4, 16 and 26 miss test members their own changes break, so each card's gate fails from the state the earlier cards leave.
    fix: >-
      Card 4: add `internal/driftengine#templateConfig` (config_test.go:224 sets BaselineModel and VerifyModel), `internal/driftengine#newVerifyFixture` (verify_test.go:175 reads cfg.VerifyInteractive) and `internal/driftengine#TestDriftVerify` (verify_test.go:869 sets VerifyInteractive) to its Edit list, each moving to the renamed field.
      Card 26: add `internal/driftengine#templateConfig` to its Edit list, which drops VerifyMaxRuns there.
      Card 16: add `internal/shedrecipe#minimalEntries` to its Edit list, whose DriftVerify row becomes `gates: [{name: judgement, attempts: 3}]` (conformance_test.go:91-96 builds it with `verify`).
  - id: F4
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/plan/19-drift-amend-defer.md:47
    summary: lym drift amend refuses an entry with ParseExpectation's message, which card 5 writes with no way-forward clause, breaking the decision record's PATTERN-refusal-way-forward constraint for a new refusal.
    fix: >-
      Card 19: Amend wraps a parser refusal as `drift amend refused: <the parser's message>; way forward: correct the entry in the --entries file and run lym drift amend <slug> again`,
      and TestAmend's row "an entry D5 refuses is refused and nothing is written" also asserts the refusal names that way forward.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/10-drift-containment-check.md:49
    summary: Check tests awaited_at at step 3 before the NotYet test at step 5, so lym drift check before Drift-Expect, where a real run has no awaited_at, reports the capture never counted and names lym drift defer instead of saying nothing is sealed yet as D6 requires.
    fix: >-
      Card 10: before step 3, a current row not past Drift-Expect by `PastRow(d.Rows, status row, "Drift-Expect")` sets Skip with "no expectation is sealed yet" and ends the check; TestCheck's row "a row before Drift-Expect" leaves awaited_at unset.
      Card 14: the TestCheck_PrintsTheFindings row with the status at Drift-Await seeds a scope record with no awaited_at, the state a real run holds there.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/08-drift-seal-baseline-checks.md:49
    summary: CheckBaseline's Unguarded state checks nothing, so on a run with no Drift-Baseline commit a missing baseline directory raises no fault and Drift-Expect fails on ListCaptures instead of the Stuck naming lym drift defer that D12 requires.
    fix: >-
      Card 8: CheckBaseline faults on a missing baseline directory whether or not a Drift-Baseline commit exists; Unguarded only skips the digest compare.
      TestSealAndBaseline gains a row where a missing directory with no Drift-Baseline commit faults naming lym drift defer.
      Card 12: TestDriftExpect's missing-baseline row runs with no Drift-Baseline commit.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/18-drift-baseline-await.md:46
    summary: Card 18 adds recipe-header text naming lym drift accept, withdraw and waive, against D14's removal of recipe-comment text and the criterion that no recipe comment names a removed verb.
    fix: >-
      Card 18: the headers say that a run parked at a Drift-Verify halt from before this change goes on with lym loom resume, which checks it under the new Drift, naming none of the removed verbs.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/09-drift-current-capture.md:47
    summary: CaptureCurrent adds an unbounded wait on the capture lock with no stated cadence, and no card adds it to PATTERN-daemon-wakeups' background list of unbounded waits.
    fix: >-
      Card 9: CaptureCurrent's lock wait polls from one second, doubling to thirty seconds while the lock stays held, as `internal/driftengine/wait.go`'s capture wait does.
      Add `pattern/PATTERN-daemon-wakeups.md` to its Edit list, listing the current capture's lock wait under "What counts".
  - id: F9
    severity: MEDIUM
    class: scope
    origin: lens:repo-invariants
    location: _lym/plan/13-drift-expect-rows.md:32
    summary: internal/loomrecipe/doc.go and docs/overview.md state the drift row order, Drift-Recheck's on_stuck route, Drift-Baseline's agent and the removed lym drift verbs, which cards 13, 16, 18 and 22 change, and no card edits either file.
    fix: >-
      Card 13: move `internal/loomrecipe/doc.go` from Uses to Edit; both recipes' row orders gain Drift-Expect.
      Card 16: add `internal/loomrecipe/doc.go` to its Edit list; drop the sentence that Drift-Recheck's on_stuck names Drift-Verify and say Drift-Recheck runs the DriftVerify engine.
      Card 22: add `docs/overview.md` to its Edit list; line 279 lists the remaining verbs `check`, `amend`, `capture`, `defer`, `status` and `capture-child`, and the drift paragraph at lines 614-619 places Drift-Expect after Drift-Await, gives Drift-Recheck the DriftVerify engine with no route back, and has Drift-Baseline record the scope from config with no agent.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/19-drift-amend-defer.md:53
    summary: Defer keeps refusing while the capture lock is held with the way forward lym loom resume, but after card 9 a lym drift check or drift gate entry also holds that lock during its current capture, where resume is not the way forward.
    fix: >-
      Card 19: Defer's refusal while the capture lock is held names waiting for the running capture or check to finish, then running lym drift defer again;
      TestDeferWaive gains a row where the lock is held at an admitted halt and the refusal names that way forward.
  - id: F11
    severity: LOW
    class: scope
    origin: handler
    location: _lym/plan/25-driftengine-retire-acceptance.md:3
    summary: CountedCapture loses its last production callers when card 16 rewrites Drift-Verify and card 25 deletes Waive, and no card deletes it or TestCountedCapture.
    fix: >-
      Card 25: add `internal/driftengine#CountedCapture` and `internal/driftengine#TestCountedCapture` to its Delete list, and drop CountedCapture's line from the driftengine doc.go it already edits.
---

### [BLOCKING] Judge snapshot treats loom's own status commits as judge edits

**Location:** `_lym/plan/15-drift-judge-seat.md:65`, `_lym/plan/16-drift-verify-lines.md:120`, `:126`

**Issue:** The run status `_lym/shed/drift-rework/status.json` is tracked on the weft (`git ls-files` in the weft lists it), and loom commits it at each row transition (`loom: Drift-Baseline -> running` and the like in the weft log).
`Changes` exempts only `judgement.yaml`, and card 16 compares a resumed row against the recorded snapshot until the trees match.
A `lym loom resume` of an interrupted judge, or of a judge-change Stuck after the operator restored the trees, moves the weft HEAD and changes `status.json` after the snapshot, so the row stops again on every resume.
The stated way forward, resetting the weft branch, would drop loom's own commits.
D9 guards changes "the session made", which loom's own commits are not.

### [BLOCKING] Duplicate delete of two verifyFixture helpers

**Location:** `_lym/plan/25-driftengine-retire-acceptance.md:48`, `_lym/plan/16-drift-verify-lines.md:73`

**Issue:** Card 16 deletes `verifyFixture.captureForAcceptance`, `agentAccept` and `sessionAccept`, and card 25 deletes `agentAccept` and `sessionAccept` again, so card 25 targets members already gone and fails `delete-target-gone` at dispatch.
The validate-plan overlay showed that dropping the pair from card 25 leaves card 25's `statusAt` re-sign caller-uncovered at `verify_test.go:227` and `:244`, and dropping only card 16's pair leaves `captureForAcceptance` caller-uncovered.
The proposed fix moves all three deletions to card 25, which the overlay admits together with every other fix in this review.

### [BLOCKING] Cards 4, 16 and 26 miss test members their changes break

**Location:** `_lym/plan/04-drift-config-keys.md:3`, `_lym/plan/16-drift-verify-lines.md:3`, `_lym/plan/26-drift-retire-skip-keys.md:3`

**Issue:** Card 4 renames `BaselineModel`, `VerifyModel` and `VerifyInteractive`, but `templateConfig` (`config_test.go:224`), `newVerifyFixture` (`verify_test.go:175`) and `TestDriftVerify` (`verify_test.go:869`) read the old names, and none is listed, so the driftengine test package fails to compile at card 4's gate.
Card 26 drops `VerifyMaxRuns`, which `templateConfig` also sets.
Card 16 makes `driftVerifyEntry` refuse anything but one `judgement` gate, while `minimalEntries` (`conformance_test.go:91-96`) still builds DriftVerify with a `verify` gate; only card 23 edits `minimalEntries`, after card 16.

### [BLOCKING] lym drift amend refuses a bad entry with no way forward

**Location:** `_lym/plan/19-drift-amend-defer.md:47`

**Issue:** Card 5 has `ParseExpectation` refuse "with no way-forward clause since the expect agent's `expectation` entry re-prompts on it".
Card 19 uses the same message as the refusal of the new `lym drift amend` verb, where no entry re-prompts the parent.
The decision record's Constraints require that "every new refusal names its way forward, and a test reaches it" (`PATTERN-refusal-way-forward`), and `TestAmend`'s row checks only that the entry is refused.

### [MEDIUM] lym drift check before Drift-Expect names defer instead of reporting nothing sealed

**Location:** `_lym/plan/10-drift-containment-check.md:49`, `_lym/plan/14-drift-check-verb.md:30`

**Issue:** `awaited_at` is unset until Drift-Await counts the capture (`internal/driftengine/records.go:77`), so at Drift-Baseline and Drift-Await `Check` stops at step 3 with "the capture never counted" before step 5's `NotYet`.
D6 says that before Drift-Expect `lym drift check` reports that no expectation is sealed yet and checks nothing.
Card 14's test seeds `awaited_at` at Drift-Await, a state a real run never holds, so it passes over the wrong path.

### [MEDIUM] CheckBaseline's Unguarded state hides a missing baseline directory

**Location:** `_lym/plan/08-drift-seal-baseline-checks.md:49`

**Issue:** `Unguarded` "checks nothing" when no commit carries `Drift-Baseline`, which is the state of a capture counted before this lands until Drift-Expect seals.
Card 12 reads D12's missing-baseline Stuck at Drift-Expect through `CheckBaseline`, so on such a run with `.lym` cleaned it goes on to `ListCaptures` of a missing directory and fails with an I/O error instead of naming `lym drift defer`.
No test row combines a missing directory with no trailer commit.

### [MEDIUM] Card 18 writes removed verb names into the recipe headers

**Location:** `_lym/plan/18-drift-baseline-await.md:46`

**Issue:** The new header note names `lym drift accept`, `withdraw` and `waive`, verbs card 22 removes.
D14 removes those verbs with "their stencil and recipe-comment text", and the Review criterion says no recipe comment names a removed verb.

### [MEDIUM] The current capture's unbounded lock wait has no cadence and no PATTERN listing

**Location:** `_lym/plan/09-drift-current-capture.md:47`

**Issue:** `CaptureCurrent` waits for the capture lock "with no bound of its own, sleeping and retrying", with no interval or backoff stated.
`PATTERN-daemon-wakeups` requires an unbounded wait to wake at most once a second and back off while idle, and its background file lists every such wait, among them the existing capture wait in `internal/driftengine/wait.go`; no card adds the new one.

### [MEDIUM] Two docs describing the drift rows and verbs are left stale

**Location:** `_lym/plan/13-drift-expect-rows.md:32`

**Issue:** `internal/loomrecipe/doc.go:15-19` gives both recipes' row orders without Drift-Expect and says Drift-Recheck's `on_stuck` names Drift-Verify; card 13 lists it only under Uses, and card 16 drops that route.
`docs/overview.md:279` lists the removed `scope`, `accept`, `withdraw`, `compare` and `waive` verbs, and lines 614-619 describe the accepted profile, the route back from Drift-Recheck and Drift-Baseline's agent; no card edits the file.

### [LOW] Defer's held-lock refusal names the wrong way forward once checks take the lock

**Location:** `_lym/plan/19-drift-amend-defer.md:53`

**Issue:** `Defer` refuses while the capture lock is held with "lym loom resume, whose row waits on the running capture" (`internal/driftengine/verbs.go`).
After card 9, `lym drift check` and the `drift` gate entry hold that lock during their current capture, where resuming the row is not the way forward.

### [LOW] CountedCapture is left with no caller

**Location:** `_lym/plan/25-driftengine-retire-acceptance.md:3`

**Issue:** Its production callers are `verify.go:170`, which card 16 rewrites to judge from the tracked record alone (Shared Decision "counted-from-the-tracked-record"), and `Waive` at `verbs.go:258`, which card 25 deletes.
Afterwards only `TestCountedCapture` and a doc.go line name it, and no card removes them.

## Rejected

- Handler hypothesis that `FindTrailerCommit` walking the weft history from HEAD would find another landed run's `Drift-Seal`, `Drift-Baseline` or `Drift-Records` commits: the weft history holds other runs only as fresh `batten: run records for <slug>` copies, with no merges and only this run's row commits, so no other run's drift trailers reach it.
- Handler hypothesis that `lym drift check` must take its cap from the keep-warm resolver (`PATTERN-keep-warm-cap`): the entry governs how a wait an agent blocks on is bounded, and the card bounds none; the capture keeps its configured timeout.
- Handler hypothesis that `TestCheck_PrintsTheFindings`' missing-baseline row duplicates `TestCheck`'s: the CLI row pins that the verb prints the fault without failing through its real wiring, which `TestCheck`'s fakes do not reach.
- Handler hypothesis that card 10's admission of a gated `Implementer` row lacks a stated bound: the card and its Shared Decision bound it by Merriam's existing rule that a turn end with an outstanding fork is a wait.
