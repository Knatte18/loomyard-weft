---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:63
    summary: D4 bound on a notice that QueueNotice only logs is garbled and calls a logged notice lost silently, and A1 and the acceptance criteria do not name the no-strand case
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:116
    summary: D7 checkout bound says an archived pair's records come into the worktree but not that they replace the worktree's own per-worktree config and run state
  - id: F3
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:79
    summary: D5 states the unreachable-origin stop for reconcile and checkout three times in its own text, and D6 restates its live-pair sentence
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:111
    summary: D7 justifies the archive-guard bypass by a present warp worktree for raw and unmanaged cases alike, and then concedes that a raw worktree proves no live pair
---

Focus questions settled against the code (`internal/orchengine/notice.go`, `internal/fabricengine/reconcile.go`, `checkout.go`, `add.go`, `origin.go`, `internal/hubgeom/parent.go`) and the decision record.

- Focus 1 (D7 `switchOrForkWeft`): the Decision states checkout adopts an origin weft whether or not an archive tag covers it, states that nothing on origin changes, and the acceptance criteria carry an archive-tag clause.
  What the stale records can change is stated only as "records" (F2).
- Focus 2 (D7 raw-warp and unmanaged branches): the bound holds as stated.
  The Decision admits that a raw worktree proves no live pair, accepts the weaker bound with its reasons, names the `raw_adopted` to `weft_recreated` change, and the acceptance criteria reach both branches, the raw one with the archive-tag and unreachable-origin rows.
  Not a defect beyond F4's wording.
- Focus 3 (D6 origin record): D6 states an adopted weft's existing record is kept, a weft with none gets one naming the acting worktree, and an acceptance clause pins it.
  Keeping a record from another machine is safe: `decideParent` forms the parent name from the recorded worktree name and the shortname and never needs the parent worktree to exist.
  Not a defect.
- Focus 4 (D4 notice): the Decision covers a `QueueNotice` error and the no-strand `queued=false` case, and A1 is consistent with the failed-read path, but the wording is wrong (F1).

### [LOW] D4's no-strand bound is garbled and A1 and the acceptance criteria do not name it

**Location:** `_lyx/discussion/decision-record.md` D4 (the `QueueNotice` sentence), A1, the Acceptance criteria line on a notice that cannot be queued.
**Issue:** The sentence "so nothing is lost silently only where an orch strand is recorded" misstates the behavior.
With no strand recorded, `orchengine.QueueNotice` logs the notice and returns `queued=false` with a nil error, so nothing is lost silently but the hub is not told either.
The sentence also merges two cases, an error and `queued=false`, that the plan writer must handle separately: an error is a warning, and `queued=false` needs no handling at all.
A1 says "a notice that cannot be queued is logged" without saying that this includes the failed-diff-read notice, so the one channel A1 chose for a failed read has no stated bound.
The acceptance criterion says only "a notice that cannot be queued".
**Fix:** Rewrite the D4 sentence as: a `QueueNotice` error is a logged warning, and `queued=false` because the orch state records no strand is accepted as is, since `QueueNotice` already logs the notice; neither changes the verdict, and the hub sees a notice only where a strand is recorded.
Make A1's last clause cover both notices and make the acceptance criterion name both cases.

### [LOW] D7's checkout bound does not say what the adopted weft replaces

**Location:** `_lyx/discussion/decision-record.md` D7 (the `Checkout` bound).
**Issue:** The bound says a checkout of a branch reusing a removed pair's slug "brings that pair's archived weft records into the worktree".
The weft branch also carries the pair's per-worktree `_lyx/config/` and run state, so after the switch the checked-out worktree's own loom configuration and run status become the stale pair's.
That is what the records can change, and the plan writer needs it to judge the bound.
**Fix:** Name the effect in the bound: the worktree's per-worktree config and run records are replaced by the archived weft's, bounded by the branch being one the operator named, origin being unchanged, and D4's Finalize notice reporting config changes at landing.

### [NIT] D5 repeats its unreachable-origin bound

**Location:** `_lyx/discussion/decision-record.md` D5 (the `Bound` line and the paragraph after it), D6 (the first two sentences after the lead sentence).
**Issue:** D5's `Bound` line says an unreachable origin now stops the verb, and the next paragraph says it again for `reconcile` and `checkout`, with the same offline-checkout example that A4 and an acceptance criterion also carry.
D6's "A local weft branch alone makes the pair live; archive tags are consulted only when there is no local weft branch" restates the lead sentence's disjunction.
**Fix:** Merge D5's two bound passages into one, keeping the offline-checkout consequence, and fold D6's restating sentence into the lead sentence.

### [NIT] D7 justifies the archive-guard bypass by a property a raw worktree lacks

**Location:** `_lyx/discussion/decision-record.md` D7 (the second sentence of the Decision).
**Issue:** The sentence adopts an origin weft "because a warp worktree on that branch is present" and says this covers the raw-warp and the unmanaged cases alike.
Two sentences later the record concedes that a raw warp worktree proves no live pair.
The justification therefore holds for the managed case and is only an accepted trade for the raw one.
**Fix:** State the justification for the managed case alone, and let the raw-worktree sentence carry its own accepted bound.

## Focus departures

None.
