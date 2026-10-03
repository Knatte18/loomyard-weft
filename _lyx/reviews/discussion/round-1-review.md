---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D8, D9
    summary: The landed rule for a remote tip counts the branch's own remote-tracking ref as another ref, so the remote deletion gate always passes and unlanded pushed work can be deleted everywhere
  - id: F2
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md D3, D6
    summary: D6's residual sibling-dirtiness refusal fires inside Remove after D4 ended the session and after D6's commit, contradicting D3's guarantee that a refused call leaves session, strands and pair untouched
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md D7
    summary: D7's bound claims the retiring mark follows a successful record commit, but the teardown command joins commit and remove with a semicolon, and the bound is silent on a mark set by hand or left by a failed remover spawn
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md D2
    summary: D2's bound covers only non-quiet drivers and never states that a parked driver's session, and its resumable conversation, is ended without a refusal
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md D5
    summary: D5's nothing-remains test ignores a stray path at the pair location, so that case returns the typed not-found error, the stray path goes unreported, and batten records done
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md Open risks
    summary: The warp-SHA source for D6's trailer is already decided but sits under Open risks, while the support log's Q13 still calls it open
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md Goal
    summary: The Goal promises no branch left on origin or locally, but D8 keeps an unlanded branch and remove without --remote leaves origin untouched
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md D10
    summary: D10 carries a dated verification narrative and board-item history that belong in the support log, which already records the check
---

### [BLOCKING] The remote landed rule has no effective bound

**Location:** `decision-record.md` D8 and D9 (D9 inherits "D8's rule").

**Issue:** D8 defines landed as "reachable from another ref, or merging it into the recorded parent's tip changes nothing (the existing `checkUnlandedWork` rule applied to the fetched remote tip)".
`checkUnlandedWork` (`internal/fabricengine/destroy.go`) runs `rev-list --count refs/heads/<b> --not --exclude=refs/heads/<b> --glob=refs/heads/* --remotes --tags`: it excludes only the local branch's own ref and counts every remote-tracking ref.
Applied to a remote tip, the branch's own `refs/remotes/origin/<warp>` holds that same tip, so the count is always 0 and the gate always passes.
This combines with today's local step into a concrete loss path: a pushed warp branch with unlanded work has its local copy deleted because the remote-tracking ref holds it (the existing check), and then D8 deletes the origin copy because the same remote-tracking ref holds it.
No ref is left holding the work.
D9 has the same hole for every leftover on origin.
D8's bound ("only with landed work") is therefore not a bound.

**Fix:** In D8, define "another ref" for a remote tip as any ref except the branch's own copies: its local branch and every remote-tracking ref of the same name.
State in D9 that the same exclusion applies.
Add an acceptance criterion: `remove --remote` on a pair whose unlanded warp branch is pushed keeps the origin copy with a reason.

### [BLOCKING] D6's residual refusal breaks D3's guarantee

**Location:** `decision-record.md` D3 and D6.

**Issue:** D3's read-only probe lists slug validation, prime, merge in progress and warp-side dirtiness, and promises "A refused call leaves the session, strands and pair untouched."
D6 keeps a no-force refusal for sibling-side changes outside the scoped pathspec, and orders it after its own commit ("Anything still uncommitted after that commit … is refused").
The composite runs `Remove` only after D4 has ended the session, so this refusal fires after the strands are gone and after D6's commit has added a commit to the sibling branch.
The two Decisions contradict each other, and an operator's interactive strands are lost exactly in the case D3 exists to protect.

**Fix:** In D6, check sibling-side dirtiness outside the scoped pathspec before the commit, so a refusal precedes every mutation.
In D3, add that check to the probe.
Narrow D3's guarantee to the race the probe cannot close: a change outside the pathspec made between probe and `Remove` is refused with the session already ended.
Add an acceptance criterion for the sibling case next to the warp-side one.

### [MEDIUM] D7's bound overstates what the retiring mark guarantees

**Location:** `decision-record.md` D7.

**Issue:** The bound says the mark "happens after the driver's teardown command has committed its records".
The teardown command is `lyx loom commit-records; lyx reed remove --name driver --detach` (`internal/loomcli/driverprompt.go`), joined with `;` so the removal runs even when the commit fails.
The ordering is loom's command string, not reed's, and it does not imply a successful commit.
Anyone else who runs `reed remove --name <driver> --detach` also marks the driver retiring, and a detached-remover spawn failure after the mark leaves a live strand marked retiring for good.
D7 states none of these, so the bound on start's synchronous removal, and on D2 counting the mark as quiet, rests on a false premise.

**Fix:** Restate the bound: the mark records that some caller requested the strand's removal, and start only completes that request.
Uncommitted records are not lost, because they stay in the sibling worktree for D6's commit or the next run's commit.
Say what happens to the mark when the remover spawn fails: clear it, or accept it, since the removal was requested anyway.

### [LOW] D2 does not bound what a parked strand loses

**Location:** `decision-record.md` D2.

**Issue:** D2 counts a parked driver as quiet, so `fabric remove` ends its session without waiting or refusing.
The bound covers only non-quiet drivers.
A parked driver is a session left open for `lyx loom start` to resume, so the remove ends a resumable run.
The `remove` call is explicit and the driver committed its records when it parked (`driverParkCommand`), so the edge is defensible, but the Decision does not say what it lets through.

**Fix:** Add to D2's bound that a parked driver's session is ended without refusal, that its records were committed by its park command, and that removing the pair already ends the run it would resume.

### [LOW] D5's not-found test ignores a stray path

**Location:** `decision-record.md` D5.

**Issue:** D5 reports a non-worktree path at the pair location in a result field and returns the typed not-found error when "nothing of the pair remains", which batten maps to done.
When the stray path is the only thing left, as in the incident D4 describes, the not-found test is satisfied, an error carries no result, and the path goes unreported while batten records the teardown as done.
The rest of the done mapping is bounded: slug validation and the prime refusal still refuse first, and the listed artifacts cover what `PairSiblingRemnant` and `RemovePairBranch` checked.

**Fix:** State that the not-found error names a stray path at the pair location when one is present.

### [LOW] A decided pick sits under Open risks

**Location:** `decision-record.md` Open risks, first bullet; `support-log.md` Question ledger Q13.

**Issue:** The first open risk already decides the warp SHA for the finish-path trailer: the local warp branch tip when present, else the last recorded `Warp-SHA` on the sibling branch.
That is a requirement for the plan writer, phrased as a risk, while Q13 still calls the question open.
The second open risk (unserialized concurrent teardowns) carries no requirement beyond D5's convergence on a re-run, and stays a risk.

**Fix:** Move the pick into D6 as part of the Decision, drop the first risk, and mark Q13 resolved.

### [LOW] The Goal overstates branch removal

**Location:** `decision-record.md` Goal.

**Issue:** "leaves no branch behind on either origin or locally" contradicts D8, which keeps a branch with unlanded work, and the existing local gate, which does the same.
Without `--remote`, origin is not touched at all.

**Fix:** Say the remove leaves no landed branch behind, and only with `--remote` on origin.

### [NIT] D10 carries narrative for the support log

**Location:** `decision-record.md` D10.

**Issue:** "Verified 2026-10-03: `origin/main-weft` tracks …" and the rationale's history ("the board item was seen before batten's prime commits started pushing") are deliberation the support log's Interview turn 11 already records.
The plan writer needs only the decision: no code change, plus one sentence in the fabricengine package doc and its reason.
D4's rationale and Auto-mode assumption 3 are judged not defects: D4's rationale states why the session ends first, and assumption 3 tells the plan writer the hand-back case needs no work.

**Fix:** Cut the verification sentence and the board history from D10, keeping the decision and the invariant it follows.
