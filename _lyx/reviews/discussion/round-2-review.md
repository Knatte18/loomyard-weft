---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 and R2
    summary: D4 and R2 rest on a false premise, because landing never merges the weft, so a resumed Finalize recomputes the same non-empty diff for every merge shape and the notice is duplicated, never lost
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 and Acceptance criteria
    summary: D4 makes an unreadable config diff halt the landing as Stuck, which departs from the loom rule that a guard not protecting correctness warns and records, and from Finalize's own rule that git faults are errors and never Stuck
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5, D7 and A4
    summary: D5 Bound says the skip opt-out is the only way to fork past an origin weft, but reconcile and checkout have no opt-out, and checkout forking a brand-new branch now needs the network; the bound and the origin-less and unreachable cases for checkout are unstated
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6
    summary: D6 live-pair rule does not say what happens under the skip-git and skip-push options, where the leftover probes do not run and D5 skips step 2
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Several stated behaviors have no assertion, namely live pair with no origin warp, reconcile adopting an archive-covered origin weft, origin-less hubs, skip options, and .weft/ on the origin-adopt paths
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 and Notes for the plan writer
    summary: A plan-actionable ordering rule, that the live-pair decision and every refusal precede the first creation, lives only in the optional Notes and not in D6
  - id: F7
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md Notes for the plan writer
    summary: Notes list creation sites, helper names and files, the exact-symbol class the writer stencil says is computed at Plan time, and repeat D4's placement rule
---

### [MEDIUM] D4 and R2 misstate what a resumed Finalize sees

**Location:** `_lyx/discussion/decision-record.md` D4 (the "resumed Finalize ... non-squash" sentence, the "which makes the base the task's real fork point" clause and the Bound) and R2.
**Issue:** `Fabric.Merge` and `Fabric.MergeIn` merge the warp checkout alone; their doc comments say the weft side is not a merge participant (`internal/fabricengine/merge.go`).
So neither Finalize's catch-up merge-in nor its parent-side merge, squash or not, ever moves the task's weft relative to the parent's.
The merge-base of the two weft branches is the fork point at every moment of the run, and the task's weft tip is never an ancestor of the parent's weft.
Consequences: the diff is identical before and after the merge-in, so the placement "before the merge-in" is not what makes the base the fork point; a resumed Finalize after a completed non-squash landing computes a non-empty diff, not an empty one; and R2's claim that a non-squash resume loses the notice is backwards.
The real behavior, for both merge shapes, is that a resume re-queues the notice, so the bound is a possible duplicate, never a lost notice.
**Fix:** Delete the non-squash sentence, state that landing never merges the weft so every resumed Finalize recomputes the same diff, restate the Bound as "a resumed Finalize that reaches Done again queues the notice again", rewrite R2 as the duplicate risk, and add one Acceptance criterion pinning the re-queue on resume.

### [MEDIUM] An unreadable config diff halts the landing

**Location:** `_lyx/discussion/decision-record.md` D4, A1 and the Acceptance criterion on a failed diff read; `_lyx/discussion/support-log.md` Q5 and Rejected alternatives.
**Issue:** The notice is advisory: it protects no correctness of the landing.
`PATTERN-refusal-way-forward` says a guard in loom that does not protect correctness warns and records rather than halts, and `Finalize.Call` already maps a git fault (the status commit) to a returned error, never to Stuck.
D4 instead makes the read failure a Stuck, and A1's own argument for the notice channel over a stop applies equally here.
**Fix:** On a read failure Finalize logs a warning, still lands, and queues one notice saying the config diff of the slug could not be read and naming the two branches so the hub diffs them by hand; the silence the bug reports stays impossible without halting a landing.
Update the Acceptance criterion, Q5, and add the Stuck alternative to the support log's Rejected alternatives.

### [MEDIUM] D5's bound is wrong for reconcile and checkout

**Location:** `_lyx/discussion/decision-record.md` D5 (Bound), D7, A4.
**Issue:** D5 says "the skip opt-out is the only way to fork past an origin weft", but only `add` carries skip options; `Checkout(l, branch)` and `Reconcile(l)` take none, and D5 itself says so two lines earlier.
For them there is no way to fork past an origin weft when origin is unreachable, and the scope forbids a new flag.
A4 covers only reconcile's network need.
`checkout` to a new branch with no weft anywhere, which today forks offline, now asks origin first and fails offline; nothing states that or what the way forward is.
D5's no-`origin`-remote skip is stated, but no assertion pins that reconcile and checkout on such a hub still fork as today.
**Fix:** State in D5's Bound that for reconcile and checkout an unreachable origin stops the pair or verb with an error naming origin, there is no opt-out, and the way forward is restoring access; extend A4 to checkout; add Acceptance rows for `checkout` with an unreachable origin (error, both sides rolled back, nothing forked) and for reconcile and checkout on a hub with no weft origin (fork as today).

### [LOW] D6 is silent about the skip options

**Location:** `_lyx/discussion/decision-record.md` D6.
**Issue:** `Add` runs the leftover probes only without `SkipPush` and `SkipGit`, and D5 makes the resolver skip origin under either; D6's live-pair rule reads origin tips and archive tags and does not say it is skipped too.
**Fix:** Add to D6 that under either option no origin is consulted, the pair is live only by a local weft branch, and the warp branch forks from HEAD as today.

### [LOW] Decisions with no assertion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.
**Issue:** Every other stated behavior, including the rollback of a fast-forwarded branch and the failed diff read, has an assertion.
These do not: a live pair whose origin has no warp branch (D6, warp forks from HEAD, weft adopted); `reconcile` adopting an origin weft that an archive tag covers (D7's stated bypass and its bound); `add` under a skip option with an origin weft present (D5); `.weft/` on the origin-adopt path (D1 names it, the add and reconcile adopt criteria do not assert it).
**Fix:** Add each as a clause or row of the nearest existing criterion, not as a new test.

### [LOW] An ordering rule lives only in the optional Notes

**Location:** `_lyx/discussion/decision-record.md` D6; Notes for the plan writer, third bullet.
**Issue:** "The live-pair decision belongs [before the first mutation], so a refusal still comes before any creation" constrains the plan, yet only an optional head-start note carries it, and D6's fast-forward and true-divergence refusal depend on it.
**Fix:** Move it into D6 as a sentence, then drop the Notes bullet.

### [NIT] Notes carry plan-time symbol lists

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer.
**Issue:** The first two bullets list creation sites, helper names and files, which the writer stencil says is computed fresh at Plan time and which a pending test-suite prune or a rename silently falsifies; the fourth repeats D4's placement.
The completeness-before-leanness test: no requirement is carried only there once F6 moves the ordering rule into D6, and D1 already states the requirement without the list.
**Fix:** Remove the section.
