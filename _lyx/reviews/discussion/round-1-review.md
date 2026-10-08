---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md Decisions 4 and 7, Acceptance criteria (lyx batten run pair)
    summary: A prime skipped as mid-merge leaves its committed config stale, so lyx batten run forks a new pair carrying retired keys, breaking the operator's no-inherited-retired-key call that Decision 4 and the acceptance criterion promise
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decisions 9 and 10, Open risks
    summary: Decision 9's bound says only the start verbs stop while a reconcile fails, but Decision 10 makes every run's Preflight Stuck too, including on a broken file in an unrelated pair, and neither Decision states it
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 14
    summary: Decision 14's role test (every role that edits files, its own artifacts included) contradicts its exclusion of review-only and judge roles, which write review, verdict, description and reflection files, so the PATTERN entry's file-editing role has no settled membership
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md Decisions 5 and 10
    summary: The hub-lock holder waits unboundedly on each pair's combined write lock, so the bounded hub-lock wait has no reconcile duration to sit above, and Preflight's lock-timeout outcome and way forward are unstated
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Decision 14 rationale, Open risks
    summary: Decision 14's rationale restates rejected alternatives already in the support log, and two Open risks restate Decision 7 and 9 bounds
---

### [BLOCKING] A mid-merge prime breaks the no-inherited-retired-key promise

**Location:** `_lyx/discussion/decision-record.md` Decision 4, Decision 7 (last paragraph), Acceptance criterion "A pair created by `lyx batten run` after a binary change carries no retired key in any `_lyx/config` file".

**Issue:** Decision 4 rests on the prime's reconciled config being committed before pair add forks the prime's local records branch.
Decision 7 skips any worktree whose records side refuses the commit as mid-merge (`ErrMergeInProgress` or foreign merge state, which `Fabric.Commit` in `internal/fabricengine/commit.go` refuses before taking any lock), leaves it unwritten, and lets the start verb continue.
The prime is a worktree in that walk, so a prime mid-merge (for example during a landing conflict) is skipped, and `lyx batten run` then proceeds to create the pair, which forks the prime's stale committed config with its retired keys.
The record never states this outcome.
The new pair's own `Preflight` would then see the still-stale stamp and reconcile the pair itself before Loom-Preflight, so the "Loom-Preflight passes" half of the criterion may still hold, but the "carries no retired key in any `_lyx/config` file" half and the Auto-mode line "a new pair never inherits retired keys (Decision 4)", an operator call, do not.
The mid-merge skip is an auto-pick and the no-inheritance rule is an operator call; the auto-pick silently overrides it.

**Fix:** Decide the prime-skip case in the record: either `lyx batten run` stops with a way forward when the prime itself was skipped as mid-merge (the pair-skip rule stays as is), or `lyx batten run` reconciles and commits the new pair right after creating it, before its driver starts.
Then make Decision 4, Decision 7 and the acceptance criterion agree on that outcome.

### [MEDIUM] Preflight's stop is missing from Decision 9's bound

**Location:** `_lyx/discussion/decision-record.md` Decision 9 ("Bound: while the stamp stays stale and failing, only the start verbs stop"), Decision 10, Open risks ("no run can start or resume in the hub").

**Issue:** Decision 10 has every loom run's `Preflight` row call the same hub-wide walk on a stale stamp and go `Stuck` on its failure.
A walk failure anywhere in the hub, such as an unparseable `batcher.yaml` in an unrelated pair, keeps the stamp stale, so every run that reaches `Preflight` (a fresh run, a `goto` re-entry) goes `Stuck`, not only the start verbs.
Decision 9's bound and the Open risk both claim the blast radius is the start verbs, which understates what the new edge can block.
Decision 10 also hands `Preflight` Decision 9's way forward, "re-run the start verb", without saying which start verb applies to a `Stuck` row (presumably `lyx loom resume`).

**Fix:** Widen Decision 9's bound and the Open risk to name `Preflight`, state that a failure in any worktree makes every run's `Preflight` `Stuck`, and name the `Preflight` row's own way forward (fix the named file, then `lyx loom resume`).

### [MEDIUM] Decision 14's role membership contradicts itself

**Location:** `_lyx/discussion/decision-record.md` Decision 14 ("Roles that get it: every spawned role that edits files, its own artifacts included — ... review-only and judge roles do not"), its PATTERN entry wording ("a file-editing role's opening stencil carries the edit-directive marker"), Acceptance criterion "Every file-editing role's rendered prompt contains the edit directive's rule".

**Issue:** The stated test counts writing a role's own artifact as editing files, which is why Master (state files), Plan-Write and Discussion-Write are in.
By that test the burler reviewer (writes its review file, `contracts/stencils/burler/burler-step-2-review.md`), the bouncer judge (verdict file), the landing describer (`landing-template-describe.md`: "Write exactly one file, `{{.description_path}}`") and the friction reflection role (`friction-template-reflection.md`: "Write the report file") all edit files, yet the Decision excludes review-only and judge roles and does not mention the describer or the reflection role.
The PATTERN entry and the stencil scan key on "file-editing role", so the plan writer cannot tell which opening stencils the scan must require the marker in.
The rule text itself does not contradict any listed role's write limits: it constrains the tool, not the write surface.

**Fix:** Replace the test with one that matches the intended set, e.g. "every role that edits files other than writing a single artifact with one Write", or name the full set explicitly (including or excluding the describer and reflection roles), and make the PATTERN entry and the acceptance criterion use the same definition.

### [LOW] The hub-lock wait has no bounded duration to sit above

**Location:** `_lyx/discussion/decision-record.md` Decision 5 ("the plan picks a wait comfortably above a hub-wide reconcile's duration"), Decision 10.

**Issue:** The holder of the hub lock commits into every pair through `Fabric.Commit`, which takes the pair's combined write lock with the blocking `lock.AcquireWriteLock` (`internal/fabricengine/commit.go`), with no wait bound.
A pair whose write lock is held (a long merge in `internal/fabricengine/merge.go` takes the same lock) stretches the walk without limit, so a waiter's bounded hub-lock wait can expire behind a healthy holder, and "comfortably above a hub-wide reconcile's duration" names a duration that is not bounded.
Decision 5 states what a timed-out start verb does; Decision 10 does not state what a timed-out `Preflight` does (`Stuck`, transient, or a retry) or its way forward.

**Fix:** State that the walk's per-pair commit wait is bounded too (and what exceeding it does: skip the pair like a mid-merge one, or fail), and state `Preflight`'s outcome and way forward on a hub-lock timeout.

### [NIT] Relocation candidates

**Location:** `_lyx/discussion/decision-record.md` Decision 14 rationale; Open risks entries "While a reconcile keeps failing ..." and "A pair that stays mid-merge keeps the stamp stale ...".

**Issue:** Decision 14's rationale argues against adding the rule to the PATTERN directive and against typing `scribe:conversation`; both are already in the support log's Rejected alternatives, and the Decision's own text (a dedicated, unconditional directive; Constraints' `PATTERN-role-skills-typed` line) already carries the constraint the plan writer needs.
The two Open risks restate Decision 9's bound and Decision 7's mid-merge retry rather than naming a risk the Decisions leave open.

**Fix:** Cut Decision 14's rationale to the one fact the Decision does not state (the rule lives in a chat skill spawned roles never load), and drop the two restating Open risks, or fold anything new in them into Decisions 7 and 9.

## Verdicts on focus entries with no finding

- Decision 12: the ancestry branch suppresses deploy advice only when the binary's revision is absent from HEAD's history, and emits it as today otherwise; the no-revision branch keeps today's classification, deploy advice included; the open object-store case is delegated with the bar "do not tell the operator to deploy", which bounds it. Not a defect.
- Rejected-alternative reasons: each operator-derived reason (start verbs only, stop on failure, committed config, no stop on a mid-merge pair) is carried by Decisions 3, 7 or 9; Q10 and Q11 are carried by Decisions 12 and 5. No missing constraint.
- `Preflight` as a second entry into a pair whose driver is running: the shed driver holds its run lock (`internal/shedengine/run.go`), not the pair's combined write lock, while a step runs, so `Preflight` committing into its own pair through `Fabric.Commit` does not self-deadlock; the remaining bound gaps are F2 and F4.
