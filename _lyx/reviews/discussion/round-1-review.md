---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D5, D3, D4)
    summary: D5's bound on appending during Darn-Write is asserted, not real; the post-append check passes on any section-complete record, so an append can land before the writer has validated and a later writer overwrite drops it silently
    fix: In D4, move the Go commit of the two files from the discussion gate's pass at handoff to the writer's first clean `lyx loom validate-discussion` on a darn run (the verb commits through `commitDiscussion`), and keep the handoff gate as the check. In D5, make the verb on a darn run refuse a record that is not yet committed or that differs from its committed copy, with the way forward to message the Darn-Write session, and restate the bound as that mechanical check instead of the writer's D3 promise. Add to D4 that the handoff gate refuses a record whose committed `Added after Discussion` entries are missing from the working copy. Add acceptance criteria for both refusals, and drop the Rejected alternatives line that dismisses refusing until commit, or reword it to say why this form differs.
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D9, D10)
    summary: D9 lets an alignment approve land a darn run with no orch turn, but the pr-gate judge measures against the decisions recorded in the record, which on a darn run are the writer's own unreviewed calls; D9's bound omits this, and with review_max_bounces 0 nothing else checks them
    fix: Extend D9 (or D10) so that on a darn run the alignment agent measures the diff against the board entry as amended by the record's `Added after Discussion` entries and treats the writer's own sections as a claim, exactly as D7 does for Darn-Review; deliver it as a told prompt value rendered by darn's wiring so the stencil still names no recipe row. Add the writer-authored record to D9's Bound sentence together with this rule, and add an acceptance criterion that a darn PR-Align prompt carries it.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7, D6)
    summary: D7 does not say what the review does with a correction the writer followed from a message but the parent never recorded (D6), so the review flags it against the board entry and the fixer reverts it, which is the webster-pack-last failure this task exists to fix
    fix: In D7, state that a deviation the change description names as an unrecorded parent correction is never fixed by revert; the review records it as a finding left open, which D8 carries into the record's `## Open risks` for the alignment agent and the parent to settle. Also name the bound on who writes the overriding entries as the verb's caller rule (parent, operator's unnamed shell, arbiter for `--by arbiter`), with `--by` an unverified label. Add the matching acceptance criterion and the rubric and `fasit` wording to the D7 criterion.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D4, D5)
    summary: "D4's ordering reason (commit before verify runs on the committed tree) and D5's commit-collision clause rest on a false premise: `_lyx` links into the weft worktree, `commitDiscussion` commits there through `CommitAnchoredPaths` under the weft write lock, and the writer's code commits and the verify gate are on warp"
    fix: In D4, drop the clause tying the commit to the verify gate's committed tree. In D5, replace the collision sentence with the real behavior - Go's commits of `_lyx/discussion/` are serialized by the weft write lock, and a commit with nothing new commits nothing without error - or drop it.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (## Constraints)
    summary: Three Constraints are plan bookkeeping or restate a standing rule rather than a requirement this design sets - the test-budget line repeats CLAUDE.md, the CLAUDE.md production-paragraph line asks for no change since that paragraph already names no recipe, and the docs list is a task list
    fix: Delete the test-budget line and the CLAUDE.md production-paragraph line from Constraints. Move the docs list to `## Notes for the plan writer` as a head start, leaving in Constraints only that the docs ride in the same commit as the code.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (## Acceptance criteria, D10)
    summary: D10 changes observable stencil text but no acceptance criterion covers it
    fix: Add a criterion that the alignment stencil's pr-gate judge text names no recipe row and reads the open risks the review segments carried over.
---

### [BLOCKING] Appending during Darn-Write has no real bound

**Location:** `_lyx/discussion/decision-record.md`, D5, with D3 and D4.

**Issue:** D5 bounds the new edge to "a record that passes the discussion check and that the writer, under D3, no longer edits".
The first half is what `discussionparser.AppendDecision` checks: section presence, not whether the writer has finished.
The second half is a stencil promise the verb cannot observe.
A writer that has all seven headings on disk but has not yet run `validate-discussion`, or is still filling in Decisions, passes the verb's check;
its next Write of the whole file then drops the appended entry, and the record still validates, so nothing refuses.
Under D4 the files are committed only at the handoff gate, after all the code is written, so this window is the whole of Darn-Write's coding phase, which is when the board entry says the correction arrived.
If `decision add` committed the entry first, D4's later commit records its deletion, and every later row reads a record without the correction.
The Rejected alternatives line "refuse until committed" was dismissed because the record is uncommitted for that whole phase;
committing at the writer's first clean validation removes that objection and turns the bound into a check.

### [BLOCKING] Darn's alignment approve is judged against the writer's own record

**Location:** `_lyx/discussion/decision-record.md`, D9 and D10.

**Issue:** `loom-template-alignment.md`'s pr-gate judge text reads "judge the run against its board entry's goal and the decisions recorded in the decision record", and counts "a recorded decision the change breaks" as a scope deviation.
On a darn run those decisions are the writer's, which D7 itself says nobody reviews before coding, and which D7 refuses to let Darn-Review use as an answer key.
D9 gives the alignment agent the power to land the run with no orch turn and lists its bound as `alignment_pr_approve`, `skip_alignment`, the snapshot and head checks and one reject.
None of those catches a writer's misread scope that the diff faithfully implements.
With `review_max_bounces: 0` the perch is off, Describe is skipped, and the only judge before landing measures the writer against the writer.
D10 rewords one clause of that judge text but leaves this one.

### [MEDIUM] An unrecorded correction still gets reverted

**Location:** `_lyx/discussion/decision-record.md`, D7 with D6.

**Issue:** D6 has the writer follow a message-only correction and name it in the change description.
D7 measures the diff against the board entry plus `Added after Discussion` entries, so a correction the parent has not recorded by review time reads as out of scope, and nothing stops Darn-Burler-Fix from reverting it.
That is the exact sequence of `webster-pack-last` (F1 BLOCKING, revert in `f1ce95ce5`).
The record does not state what the review may let through here or what happens to the deviation.
The second half of the focus question, an entry by a non-authoritative author, is bounded by the verb's existing caller rule in `internal/loomcli/decision.go`, but D7 relies on it without naming it.

### [LOW] D4 and D5 reason about a commit race that cannot happen

**Location:** `_lyx/discussion/decision-record.md`, D4 and D5.

**Issue:** In this worktree `_lyx` is a link to `claimed-scope-edit-weft/_lyx`.
`commitDiscussion` in `internal/loomcli/wiring.go` commits through `fabricengine.CommitAnchoredPaths`, which commits in the weft worktree under the weft write lock and reports an empty commit as not committed rather than an error.
The writer's commits are code commits on warp, and the verify gate runs on warp, so D4's "before the verify gate runs on the committed tree" orders against nothing, and D5's "a commit that collides with the writer's own commit" describes a case that does not occur.
The plan writer would design an ordering or a retry the code does not need.

### [LOW] Constraints carry bookkeeping, not requirements

**Location:** `_lyx/discussion/decision-record.md`, `## Constraints`.

**Issue:** The test-budget line restates `CLAUDE.md`'s Test budget rule, which binds every task.
The `CLAUDE.md` production-paragraph line asks that a paragraph stay recipe-neutral, but that paragraph already names no recipe, so it constrains nothing.
The docs list is a to-do list for the plan, useful as a head start but not a design constraint.
Nothing here is lost by the move: the one requirement, docs in the same commit, stays in Constraints.

### [LOW] D10 has no acceptance criterion

**Location:** `_lyx/discussion/decision-record.md`, `## Acceptance criteria`.

**Issue:** Every other Decision with an observable change has a criterion; D10's stencil wording change has none.
No criterion asserts something no Decision decides.

## Focus verdicts not carried by a finding

On the first directive, D6 and the rejection-respawn path are consistent with D3's no-edit rule: D6 sends the writer to the parent rather than into the record, and a respawn rewrites only an absent or invalid record;
the defect is the window F1 describes.
