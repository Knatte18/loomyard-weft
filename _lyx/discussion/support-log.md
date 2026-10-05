# Support log — test-writing-rules

## Interview

The session ran autonomously: no operator answered questions, and every decision below is an auto-pick except where the parent's relayed scope decision settled it.

1. Read the board entry (`test-writing-rules`) and the dependent entries `test-suite-prune` and `lyx-skills-and-stencils`.
   `test-suite-prune` owns measuring, deleting, speeding up and budgeting; this task owns only the rules.
2. Explored the test-writing surface: `loom-plan-spec.md` (card criterion 3, card granularity, worked example), `loom-template-plan.md` (card rules, Create-test-file "normal shape", Step 0's `scribe:testing` load), the Plan-Review and Webster-Review rubrics (interpolated by both Bouncer and Burler), the Burler review and fix steps, the pattern directives, `webster-body-verify-fix.md`, `claims_test.go`, and `scribe:testing` in the plugin cache.
3. Mid-session, `ly:orch` relayed an operator scope decision: the task may change card-granularity rules and PATTERN entries; audit PATTERN for per-item test mandates such as `PATTERN-refusal-way-forward`; add a Testing PATTERN entry for the rule; do not edit `scribe:testing`, only name what it needs; the budget guard belongs to `test-suite-prune`.
   This matched the direction already taken and settled D3, D8 and the scribe question.
4. PATTERN audit: grepped every entry and background file for test obligations.
   Only `PATTERN-refusal-way-forward` mandates a test per item ("its reaching test"), echoed by `refusal-spec.md`.
   `PATTERN-ref-shape-registry`'s meta-tests are a fixed set of registry obligations, not per-item tests.
5. Chose PATTERN as the rule's single home because the pattern directives inline PATTERN.md into every designer, implementer, judge and review-fix prompt, and the review-fix directive already makes each violation BLOCKING.
6. Wrote the decision record and this log.
7. After delivery, `ly:orch` proposed, for the operator, a revision of PATTERN's Testing section before its verdict: P1 test-economy first and build entries moved out, P2 a time-seam rule for every tier, P3 a parallelism rule, P4 D8 agreed, P5 fakes before a real hub.
   All five agreed; recorded as D1 (P5 clause), D10 (P2), D11 (P3) and D12 (P1, plus the narrowed `PATTERN-hubforge-fixtures` line).
   The parent also asked for PATTERN.md and `pattern/` to be edited directly in this worktree; declined pending its answer, see Q10.

## Rejected alternatives

- **Rule stated in `loom-plan-spec.md` only, rubrics pointing at the spec.** Rejected: the spec reaches the plan writer and reviewers but not Webster's implementers or sessions outside loom; PATTERN reaches all of them.
- **Rule restated in each stencil and rubric.** Rejected by `PATTERN-producer-pointer-rule`, and it would drift.
- **A Go check that detects redundant tests.** Rejected: redundancy needs per-test coverage, which is `test-suite-prune`'s tool; building it here duplicates that task.
- **A lower severity for redundant-test findings than other PATTERN violations.** Rejected: the review-fix directive treats every PATTERN violation as BLOCKING, and a carve-out would weaken it for one entry; the named-covering-test requirement bounds false positives instead.
- **Rewriting the worked example to extend an existing test file.** Rejected: the example is byte-consistent with the planparser golden fixture, and changing both buys nothing; only its descriptive prose changes.
- **Dropping the reaching-test obligation for refusals.** Rejected: the obligation is sound, only its one-test-per-row granularity is the problem.
- **Editing `scribe:testing` in this task.** Rejected by the CLAUDE.md worktree rule and the operator's scope decision.
- **A Discussion-Review rubric bullet.** Rejected: Discussion writes no tests.
- **A new test pinning the new PATTERN entry or rubric bullets.** Rejected under the task's own rule: `claims_test.go`'s table and `internal/pattern`'s overview check already cover stencil wording and PATTERN format.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Where does the rule live?** Resolved by auto-pick: a new PATTERN entry `PATTERN-test-economy` with a background file (D1); confirmed by the parent's relayed operator decision.
- **Q2 — Does card granularity change?** Resolved by the operator decision relayed by `ly:orch`: yes; D3 keys granularity on public surface and cross-package reuse.
- **Q3 — Which PATTERN entries mandate a test per item?** Resolved by audit: `PATTERN-refusal-way-forward` and `refusal-spec.md` only (D8).
- **Q4 — Does `scribe:testing` need the same change?** Resolved: yes; it is out of this repo, so the needed text is named in the decision record's notes and not edited (operator decision).
- **Q5 — Severity of a redundant-test finding?** Auto-pick: BLOCKING through the review-fix directive like every PATTERN violation, bounded by naming the covering test.
- **Q6 — Change the worked example and golden fixture?** Auto-pick: no; only the descriptive prose.
- **Q7 — Change the Discussion-Review rubric?** Auto-pick: no; change the Discussion stencil's acceptance-scenario sentence instead (D7).
- **Q8 — Add a Go test for the new wording?** Auto-pick: no new test function; rows in `claims_test.go` (D9).
- **Q9 — Budget guard?** Resolved by operator decision: out of scope, `test-suite-prune`.
- **Q10 — Edit PATTERN.md in this worktree during Discussion?** Resolved: `ly:orch` relayed the operator's explicit instruction (option 3), with the driver named as committer; recorded as D13.
  The agent wrote only `PATTERN.md`, `pattern/PATTERN-test-economy.md` and `pattern/PATTERN-refusal-way-forward.md`, ran no git, and `go test ./internal/pattern/` passed.
  Reasoning before the instruction arrived:
  The launch prompt bars this agent from repository files and git; an uncommitted PATTERN.md would leave the task tree dirty, which `PATTERN-verified-tree` refuses to verify, and would be swept into an arbitrary card commit.
  The exact PATTERN text is in D1, D10, D11 and D12, so Plan can land it as the first card, or the parent can commit it directly if the operator wants review rounds to read it first.
- **Q11 — Background files for the time-seam and parallel entries?** Auto-pick: none; each line states the whole rule.
