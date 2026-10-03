---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/09-pairteardown-composite.md:48
    summary: Card 9 declares a pair with a gone task worktree quiet, so fabric remove with RefuseWhenBusy ends a possibly live driver with no wait, while its Bound claims it never ends a non-quiet driver; the exception to D2's bound is unstated and unbounded
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/09-pairteardown-composite.md:48
    summary: Card 9 makes an absent reed session mean the whole pair is quiet, dropping D2's run-lock-free clause for a run driven outside a reed session
  - id: F3
    severity: MEDIUM
    location: _lyx/plan/11-batten-teardown-through-composite.md:23
    summary: Probe refusals now surface from batten's Shutdown closure, which card 11 does not pass through teardownRefusal, so batten would report fabric's sibling-dirt refusal whose card-1 remedy names --force
  - id: F4
    severity: MEDIUM
    location: _lyx/plan/11-batten-teardown-through-composite.md:35
    summary: Card 11's bound says batten's teardown gains no new deletion, but Remote true inherits card 3's origin task-branch deletion and card 2's finish-path deletions; the Intent states neither nor what bounds them
  - id: F5
    severity: MEDIUM
    location: _lyx/plan/06-reed-end-session-by-name.md:18
    summary: EndSessionByName is a new exported lock-free kill path with no Bound in its Intent stating what it skips, which sessions it can reach and who may call it
  - id: F6
    severity: LOW
    location: _lyx/plan/11-batten-teardown-through-composite.md:3
    summary: taskWorktreePresent's doc comment (internal/battencli/wire.go:97-111) names PairSiblingRemnant and says the teardown halves use it; no card edits it, and card 12's build/vet and card 13's tripwire cannot catch a comment
  - id: F7
    severity: LOW
    location: _lyx/plan/09-pairteardown-composite.md:79
    summary: No card tests the acceptance criterion that a driver retiring during the wait has its committed stop report in the archive tag; card 9's only retiring test uses fakes
  - id: F8
    severity: LOW
    location: _lyx/plan/07-reed-retiring-mark.md:18
    summary: D7 says reed's strand status exposes the retiring flag, but card 7 adds it only to the engine type and not to the lyx reed status envelope
---

### [BLOCKING] Gone task worktree counted as quiet on the `fabric remove` path

**Location:** `_lyx/plan/09-pairteardown-composite.md:48`, with the Bound at line 67.

**Issue:** D2 defines quiet as the run lock being free and the driver strand being absent, dead, retiring or parked, and bounds the wait: it "never kills a non-quiet driver on the `fabric remove` path".
Card 9 adds "A pair whose task worktree is gone ... is quiet", so `fabric remove` (with `RefuseWhenBusy`) skips the wait and ends the session by name, including a live driver strand in it.
Neither quiet input can be probed in that state: `shedrun.RunLock` lives under the task worktree's `.lyx/shed/` (`internal/shedrun/paths.go:75`), and reed's strand table lives under the task worktree too, so both are gone with it.
The card's own Bound ("never ends a non-quiet driver when `RefuseWhenBusy` is set") then holds only because the definition of quiet was widened, and the Intent never says what this lets through.
Probing through the session by name is no remedy here: tmux can list panes but cannot tell the driver strand from any other, and a refusal would name `lyx reed down` in a pair that no longer exists, so the operator could not follow it.
D4 settles that the session is ended by name when the worktree is gone; what the card lacks is the stated, bounded exception to D2.

**Fix:** Keep the behavior and state it in card 9's Intent and Bound as a bounded exception.
It lets through ending a driver strand that may still be live, without a wait, when the task worktree is gone, on both caller paths.
It is bounded by: the state is decided as "the task worktree path is not a registered linked worktree", the same test card 2 uses for `stray_path`; the session is ended only by its exact name; and nothing such a driver writes after its worktree is gone can reach the archive, since its sync and commit resolve from the gone worktree and its writes are the D4 recreation bug.
The records already in the sibling worktree are committed before the archive by D6.

### [MEDIUM] Absent reed session drops the run-lock clause

**Location:** `_lyx/plan/09-pairteardown-composite.md:48`

**Issue:** The same sentence makes a pair "whose reed session is absent (`reedengine.ErrNoSession`)" quiet outright.
D2 requires the run lock to be free in addition to the strand clause.
An absent session answers only the strand clause (no strand can be live without a session); a run lock held by a shed run driven outside reed would be ignored.

**Fix:** State that an absent session satisfies the strand clause only, and that the run-lock probe still runs whenever the task worktree is present.

### [MEDIUM] Batten's probe refusals bypass `teardownRefusal`

**Location:** `_lyx/plan/11-batten-teardown-through-composite.md:23`

**Issue:** Card 9 runs the refusal probe inside `EndSession`, so batten's `Shutdown` closure now receives `ErrPairSiblingDirty` (and every other probe refusal) before `Remove` is ever called.
Card 11 has `Shutdown` return `EndSession`'s result and rewords `teardownRefusal`'s sibling-dirt arm, but `teardownRefusal` is applied only in the `Remove` closure (`internal/battencli/wire.go:447-450`).
Card 1 rewrites fabric's own message to name `--force` as the way past, so batten would surface exactly the remedy `teardownRefusal` exists to suppress.
`TestTeardownRefusal_RecordsRemedyNamesCommitRecordsNeverForce` calls `teardownRefusal` directly and would not catch it.

**Fix:** Card 11's `Shutdown` passes `EndSession`'s error through `teardownRefusal` as well, and a `wire_test.go` case or the existing remedy test covers the Shutdown path.
Since the probe also runs with the task worktree gone, the reworded arm names the pair from prime rather than requiring a task anchor that may not exist.

### [MEDIUM] Card 11's "no new deletion" bound is inaccurate

**Location:** `_lyx/plan/11-batten-teardown-through-composite.md:35`

**Issue:** batten calls `Remove(location, slug, false, true)` today (`internal/battencli/wire.go:447`), and card 11 keeps `Remote: true`.
After card 3 that call also deletes the landed task branch on origin, and after card 2 a half-removed pair batten used to refuse by name (`PairSiblingRemnant`) now has its sibling worktree, portal and launchers removed and its records committed.
The decision record licenses both (D1, D5, D8), but card 11 states the opposite, so its Bound misleads the attack-surface check.

**Fix:** Restate card 11's Bound: batten's teardown now also deletes the landed task branch on origin (bounded by card 3's gate: only `BranchPrefix + slug`, landed, leased) and finishes a half-removed pair (bounded by card 2: only what the full path deletes); it still never forces and ends a live driver only after its configured grace.

### [MEDIUM] `EndSessionByName` carries no Bound

**Location:** `_lyx/plan/06-reed-end-session-by-name.md:18`

**Issue:** The card adds an exported function that kills a tmux session, and possibly the hub's tmux server, without reed's op lock and without touching reed state.
The Intent explains why it exists and asks for a doc-comment warning, but states no Bound: what it skips, which sessions it can reach, who may call it.

**Fix:** Add a Bound: it skips the op lock and the state write only because both lived under the gone worktree; it reaches only the exact-match session it is named on the socket it is given, and kills the server only when no session remains; its one caller is `internal/pairteardown`, which calls it only when the task worktree is not a registered worktree.

### [LOW] Stale `taskWorktreePresent` doc comment left unassigned

**Location:** `_lyx/plan/11-batten-teardown-through-composite.md:3` (Edit list)

**Issue:** `internal/battencli/wire.go:97-111` says Worktree-Teardown's two halves use `taskWorktreePresent` and that the half-removed state "is PairSiblingRemnant's own question to answer".
After card 11 neither holds, and after card 12 the named symbol is gone.
No card edits that doc comment; card 12 checks only build and vet, and card 13's tripwire scans only for `RemovePairBranch`.

**Fix:** Add `internal/battencli#taskWorktreePresent` to card 11's Edit list and say its doc comment drops the teardown and `PairSiblingRemnant` references; also say the comment above `Teardown:` in `wire` is rewritten for the composite.

### [LOW] Retire-during-wait archive criterion has no test

**Location:** `_lyx/plan/09-pairteardown-composite.md:79`

**Issue:** The acceptance criterion "`fabric remove` on a pair whose driver retires during the wait proceeds once it is retiring, and the archive tag contains the driver's committed stop report" is carried only by card 9's fake test, which proves the proceed and not the archive content.

**Fix:** Add a `teardown_integration_test.go` case: the stub driver strand's report is committed on the sibling branch, the strand is marked retiring through `MarkRetiring` during the wait, `Run` proceeds, and the archive tag contains the report.

### [LOW] `lyx reed status` does not expose the retiring flag

**Location:** `_lyx/plan/07-reed-retiring-mark.md:18`

**Issue:** D7: "reed's strand status exposes the flag".
Card 7 adds `Retiring` to `reedengine.StrandStatus`, but `lyx reed status` builds its strand maps field by field (`internal/reedcli/status.go:42-49`), so the operator-facing status never shows it.

**Fix:** Card 7 edits `internal/reedcli#reedCLI.statusCmd` to add a `retiring` key to each strand, with a test.

## Focus departures

None.
The card 13 granularity directive was reached and judged not a defect: the tripwire fails until cards 10 and 11 move batten and `fabric remove` off `Topology.Remove`, so it cannot land with card 9, and it enforces the invariant card 13 writes.
The detached-remover and batten-grace acceptance criteria are carried by the Intents of cards 8 and 11 and were not promoted.
