---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D7
    summary: D7 makes checkout adopt an origin weft with no stated archive-guard bound, so checkout can revive a removed pair's archived weft
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D7
    summary: D7's bound for reconcile rests on a live warp worktree, but a raw warp worktree is unmanaged and proves no live pair; the raw path also changes from dormant raw_adopted to a wired weft_recreated unstated
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:D6
    summary: D6 rewrites the adopted weft's origin record to name the acting worktree as parent, which replaces the recorded parent of a pair moved from another machine, with no bound stated
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:D4
    summary: D4 leaves open what Finalize does when QueueNotice fails or queues nothing for lack of an orch strand, so the claim that nothing is lost silently does not hold
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:Acceptance criteria
    summary: no acceptance clause for D7's reconcile unmanaged path (a managed warp worktree with only an origin weft) or for checkout onto an archive-covered origin weft
---

### [MEDIUM] Checkout adoption of an origin weft has no archive-guard bound

**Location:** `decision-record.md`, D7 (`Checkout`'s `switchOrForkWeft`) and D5's bound.
**Issue:** D5 and D7 make `checkout` switch to an origin-tracked weft whenever `origin/<branch>-weft` exists.
The archive guard is argued only for `reconcile` (its warp worktree is already on the branch) and for `add` (the live-pair rule).
`checkout` has neither: a checkout of a branch whose name matches a removed pair's archived origin weft now adopts that weft, where today it forks a fresh one.
That is an edge that lets a stale weft through, and the Decision states no bound for it.
**Fix:** state in D7 whether `checkout` adopts an archive-covered origin weft; the narrowest bound is that it does, because `checkout` creates no pair and only switches an existing worktree onto a local branch tracking content already on origin, and say what the stale weft's records can change.
Add the matching acceptance clause.

### [MEDIUM] D7's reconcile bound does not hold for a raw warp worktree

**Location:** `decision-record.md`, D7 and the reconcile acceptance clause.
**Issue:** D7 bypasses the archive guard "because a live warp worktree on that branch is present" and bounds it to a slug whose warp worktree exists locally.
`reconcileMissingWeft`'s raw-warp branch (`isRawWarpWorktree`) is by definition a worktree created outside lyx, so its presence on a branch proves no live pair: a user's raw worktree on a branch that reuses a removed pair's slug would adopt the stale weft and have junctions wired into it.
D7 also turns that path from today's dormant `raw_adopted` (fresh weft, wired by the next pass) into an immediate `weft_recreated` with full wiring, and does not say so.
**Fix:** state in D7 that the bound covers the raw path, what the archive-covered adoption into a raw worktree can change, and that an origin weft turns the raw outcome into `weft_recreated`.

### [MEDIUM] D6 overwrites the recorded parent of a pair adopted from another machine

**Location:** `decision-record.md`, D6, the origin-record sentence.
**Issue:** D6 writes and commits the origin record on the adopted weft "naming the acting worktree as parent".
A live pair adopted from origin already carries its own record, with the parent it was created under; `WriteOrigin` replaces it unconditionally.
For a stacked task moved between machines this rewrites the parent that `ReadOriginFor` consumers (`hubgeom.ResolveParent`, `Remove`'s reachability check) read, to whichever branch the acting worktree is on.
The Decision states no bound for this.
**Fix:** state in D6 that an adopted weft that already carries an origin record keeps it, and that only a weft with no record gets one naming the acting worktree; add an acceptance clause.

### [LOW] D4 leaves the notice's own failure open

**Location:** `decision-record.md`, D4 and A1.
**Issue:** `orchengine.QueueNotice` can return an error and can return `queued=false` with no error when the orch state records no strand, in which case the line is only logged.
D4 says nothing about either, and A1 and Q5 claim a failed diff read is never lost silently.
**Fix:** state in D4 that a `QueueNotice` error is a logged warning and never changes the verdict, and that a notice logged but not queued is the accepted bound; align A1's wording.

### [LOW] Missing acceptance clauses for two stated behaviors

**Location:** `decision-record.md`, Acceptance criteria.
**Issue:** D7 says the origin weft is adopted before reconcile's raw-warp and unmanaged branches, but the criteria reach only the raw-warp case; and D7's checkout adoption has no clause for an archive-covered origin weft.
**Fix:** add one clause per case, extending the existing reconcile and checkout clauses rather than adding new ones.

## Settled focus questions

- Resume base and bound (focus 1): `Fabric.Merge` and `Fabric.MergeIn` merge the warp checkout alone (`merge.go` header, `fabricengine/doc.go` on the weft never being a merge participant), so the weft merge-base stays the task's fork point for squash and non-squash landings alike; D4 and R2 are correct, and a resumed Finalize that reaches Done queues a duplicate, never loses one.
- Failure semantics of a failed diff read (focus 2): warn, land, notice is consistent with `PATTERN-refusal-way-forward`'s warn-and-record rule and with Finalize mapping git faults to an error rather than Stuck; the open part is the notice's own failure, filed as F4.
- Per-verb bound of D5 and D7 (focus 3): holds for `add` (skip opt-out) and for the unreachable-origin and no-origin-remote cases; the gaps are F1 and F2.
- Skip options in D6 (focus 4): stated correctly, since `Add` runs the leftover probes only when neither option is set; no finding.
- Placement of ordering rules (focus 6): the live-pair decision and refusals before the first creation live in D6, the diff read before any parent-side mutation in D4; no finding.
