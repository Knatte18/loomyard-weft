---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4 and R2
    summary: D4 promises the notice on the already-landed path, but the merge-base diff is empty there for a non-squash landing and after the catch-up merge-in, and the read position is unstated; R2's recovery claim is false
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D6
    summary: D6 does not say which rule wins when a local weft exists and origin's weft is archive-tagged, and the archive-refusal criterion omits its no-local-weft precondition
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6 Bound and R3
    summary: D6's Bound claims a removed pair's stale warp branch is not resurrected, but R3's case resurrects it with nothing bounding it, and D6 never states that the unarchived-weft refusal is retired
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5 and A4
    summary: D5 honors skip options reconcile and checkout do not have, and treats a repo with no origin remote as a failed probe, which would newly break reconcile and checkout there
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D7
    summary: D7 bypasses the archive guard in reconcile and states a reason but no bound, and does not say why it does not conflict with D6's replace-archived rule
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D6
    summary: D6 does not say what rollback does to a pre-existing local weft branch it fast-forwarded
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Decisions D2 Detail text, D4 Stuck-on-read-failure and D6 fast-forward rollback have no criterion, and the archive criterion is ambiguous about the local weft
  - id: F8
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Added after Discussion
    summary: The added section mixes a plan-actionable rule with a dated Webster scheduling hold and rationale the Plan producer cannot act on
---

Focus verdicts that are not findings:

- D1, D2 and D3 close the reported bug without a gap.
  The weft worktree creators in the touched package are the fork, drop-runs fork, add adopt and reconcile adopt sites; the other `worktree add` sites (`boardweft.go`) build hub-level board worktrees, which carry no origin record and are never read through `ReadOriginFor`.
  A hand-made or moved worktree is the remaining route to the new error, and D2's reconcile is its repair route.
- D3's `Remove` fallback is unchanged in effect: a read error and a "not found" both skip the origin-derived parent branch, so it needs no criterion.
- R4's reviewer-resolution change is bounded by D1, D2 and the named way forward.
- D5's all-pairs behavior is fine: `Reconcile` records a per-pair `Error` and continues the loop, so one pair's resolver error does not stop the others.

### [BLOCKING] D4's already-landed path cannot produce its notice, and R2 rests on that

**Location:** `_lyx/discussion/decision-record.md` D4, R2.
**Issue:** D4 diffs the task's weft tip against the merge-base of the task's weft branch and its parent's weft branch, and queues the notice "on the merge path and on the already-landed path alike"; R2 relies on the resume reaching the already-landed path to re-queue a lost notice.
Finalize's catch-up merge-in (`mergeInStep`) runs before the parent-side merge, and the already-landed path is a resumed run: it re-runs the merge-in first.
After a non-squash landing the task branch is an ancestor of the parent, so the merge-base equals the tip and the diff is empty.
After the merge-in on a resume (squash or not) the parent is an ancestor of the task branch and holds the same content, so the diff is empty again.
A transient parent push failure is the documented re-step (`pushParent`), so this is the ordinary resume, not a rare crash.
D4 also never says where the read sits relative to the merge-in, which decides whether the base is the real fork point (before) or the parent's tip (after), and it calls the base SHA a "fork-point".
**Fix:** State that the diff is read at the start of Finalize, before the catch-up merge-in, so the base is the fork point; state that a landing that already completed with a non-squash merge, or any resume whose read sees an empty diff, queues nothing; reword the notice to carry the merge-base and tip SHAs; and restate R2 as an accepted loss on those resumes instead of a covered one.

### [MEDIUM] D6 leaves the local-weft versus archive-tag precedence open

**Location:** `_lyx/discussion/decision-record.md` D6, Acceptance criteria.
**Issue:** D6 makes a pair live when the weft exists locally, yet the non-live paragraph says an archived remote weft is replaced.
Both can describe one state (local weft present, origin weft archive-tagged), and the Decision does not say which wins.
The code today consults archive tags only when no local weft exists, and the fast-forward rule would otherwise move a local branch to an archived tip.
The criterion "origin's weft is covered by an archive tag and origin's warp diverges: today's refusal is unchanged" omits that no local weft exists.
**Fix:** State in D6 that a local weft branch alone makes the pair live and archive tags are consulted only when there is none, with the bound that `Remove` deletes the pair's local branches so a surviving local weft is the moved-back case.
Add the no-local-weft precondition to the criterion and add the local-weft case.

### [MEDIUM] D6's Bound contradicts R3 and leaves the retired refusal unstated

**Location:** `_lyx/discussion/decision-record.md` D6 Bound, R3.
**Issue:** The Bound says tying warp adoption to a live weft "keeps a removed pair's stale warp branch from being resurrected", and that a removed pair's leftovers still hit today's refusals.
R3 admits an unarchived remote weft of a removed pair, whose archive step failed, now makes the pair live, so its remote warp branch is adopted as well; R3 names only the stale run records.
Nothing bounds that case.
D6 also removes the unarchived-weft refusal (`leftoverUnarchivedWeft` and its message), a guard removal it never states.
**Fix:** In the Bound, state that the unarchived-weft refusal is retired and that an unarchived remote weft and its warp branch are adopted together; in R3, add that the warp branch returns with it, that nothing on origin is replaced, and that a plain `lyx fabric remove` archives the pair again.

### [MEDIUM] D5's skip options and origin-less repos

**Location:** `_lyx/discussion/decision-record.md` D5, A4.
**Issue:** D5 says the resolver honors "the verbs' existing skip-git and skip-push options".
Only `add` carries them; `Reconcile` and `Checkout` take no options, so the sentence is false for two of the three verbs, and the opt-out's effect (a silent fork, the reported bug) is unbounded.
D5 also makes every probe failure an error, but a repo with no `origin` remote is a supported state elsewhere in the package (`cleanup`, the warp binding's skipped outcome), and `ls-remote origin` fails there.
`reconcile` and `checkout` would newly fail on such a hub where they fork today.
**Fix:** State that `add` passes its skip options and the resolver skips step 2 when either is set, leaving today's fork behavior for that opt-out, and that `reconcile` and `checkout` always consult origin.
State that a weft repo with no `origin` remote skips step 2, and only an unreachable or failing origin is an error.

### [LOW] D7's guard bypass has a reason but no bound

**Location:** `_lyx/discussion/decision-record.md` D7.
**Issue:** "reconcile adopts an origin weft whether or not an archive tag covers it" bypasses the archive guard; the clause "because a live warp worktree on that branch is present" is a reason, and the Decision does not say what it lets through or why it does not clash with D6's replace-archived rule for the same slug.
**Fix:** Add a Bound: reconcile only creates a local branch from origin and pushes or replaces nothing, and only for a slug whose warp worktree already exists locally on that branch, so no removed pair is revived without a live local warp worktree; add's replace-archived rule applies only where add itself creates the pair.

### [LOW] D6 is silent on the rollback of a fast-forward

**Location:** `_lyx/discussion/decision-record.md` D6.
**Issue:** D6 says rollback deletes only branches this Add created, but a fast-forwarded local weft branch pre-exists; the Decision does not say whether a failed Add rewinds it.
**Fix:** State that a fast-forwarded branch is not rewound, since it only moved to a tip origin already holds, and cover it in an acceptance criterion.

### [LOW] Acceptance criteria miss four behaviors

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.
**Issue:** No criterion covers D2's Detail text and unchanged action, D4's Stuck on a failed diff read with the parent untouched, or the fast-forward rollback; the archive criterion is ambiguous (see the D6 precedence finding).
**Fix:** Extend the reconcile `.weft/` criterion with the Detail and unchanged-action assertions, add a Finalize criterion for a failed diff read, extend the fast-forward criterion with the failed-Add case, and fix the archive criterion as above.
Each is an assertion in an existing criterion, not a new test.

### [LOW] The added section mixes an actionable rule with scheduling

**Location:** `_lyx/discussion/decision-record.md` "Added after Discussion".
**Issue:** Of its text, "cards name production code as main targets and test files by path rather than by test-function glyph" is something the Plan producer acts on.
"Webster starts only after test-suite-prune-core and test-suite-prune-rest have both landed", the sync and re-validation step, and the operator-schedule rationale are orchestration the Plan producer cannot act on, name perishable task slugs and a date, and belong in `support-log.md`.
The heading also reads as history instead of a numbered Decision.
Completeness-before-leanness: the plan-actionable rule moves into a numbered Decision first; only the scheduling narrative moves.
**Fix:** Replace the section with `### D9 — Plan test targets by file path` (Decision: cards name production code as main targets and test files by path, not by test-function glyph; Rationale: a test-suite prune in flight can rename or delete test functions, a file path survives), and record the Webster hold, the sync and re-validation step and their rationale in the support log's Interview.
