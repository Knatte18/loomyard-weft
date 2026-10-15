# Support log — discussion-relevant-list

## Interview

### Turn 1 — exploration

Autonomous run with no advisors.
Read the board entry, the board list and the related notes it names (`fan-master-preread`, `review-inline-code`) by brief.
Read the discussion spec, the Discussion and Plan stencils, the rework stencil, the darn writer stencil's discussion lines, `loomengine`'s `DiscussionTable`, `PlanTable` and path accessors, `discussionparser.Validate`, the `loomcli` users of the discussion paths, and `tools/tokencount`'s header.
Findings that shaped the design: the Discussion-Write commit seam commits the whole `_lyx/discussion/` directory; the lead seat's `Outputs` drive the completion signal; the darn writer reads the same discussion spec but has no Plan step; the Discussion stencil's Step 2 forbids gathering signatures and `file:line` citations.

### Turn 2 — self-picked batch

Scope, format, check, Plan's read, review and darn questions, each settled as an auto-pick in the question ledger.

### Turn 3 — measurement, moved out of the record after lens review

The before figures are the board entry's `## Seen` section (10 runs before 2026-10-15).
The after figures come from `tools/tokencount/tokencount.sh` over the next several full loom runs, compared by the orch, with no tokencount change.
tokencount's per-role report may not show the plan lead's peak context directly; the orch then reads it from the transcripts, as it did for `## Seen`.

### Turn 4 — lens review

Six forks returned reports; none returned empty.

- scenarios, premises, pattern-fit (BLOCKING/MEDIUM, merged): `discussionparser.Validate` is shared by the Discussion-Write gate, the Discussion-Burler-Fix gate, darn's validation and `AppendDecision`; the record left the check's site open and did not name `PATTERN-gate-self-check-parity`. Fixed: D3 adds a separate check function outside `Validate`, called by the loom gate and the verb's loom branch; absence is not a finding; constraint named; criteria for darn, `decision add` and absence added.
- scenarios (MEDIUM): in-flight runs past Discussion-Write at deploy. Fixed by D3's absence tolerance.
- scenarios (MEDIUM): a relaunched writer is told only about the two reviewed files. Fixed: D1 and Scope name the relaunch wording.
- scenarios (MEDIUM): invalid input undefined. Fixed: D3 pins nested bullets, stray lines, absolute or `..` paths, empty reason, duplicates.
- scenarios (LOW): empty list legitimate for a docs-only task. Rejected: such a task still reads files; assumption reworded.
- scenarios (LOW): list may point Plan outside the worktree. Fixed: relative-path rule in D3 with its bound.
- scenarios (LOW): chair stencil's output listing not pinned. Rejected: it derives from `Outputs`, which the first criterion pins.
- premises (LOW): `loom-plan-spec.md` names nothing Plan reads. Fixed: moved to Scope Out, criterion dropped.
- premises (LOW): spec must say darn does not write the list. Fixed in D2.
- pattern-fit (LOW): `doc.go` declarer sentence needs the new accessor. Fixed in Scope.
- pattern-fit (LOW): completion signal and refusal-way-forward fit; no action.
- board-fit (MEDIUM): `engine-model-choice` rewrites `discussiontable.go`. Fixed: Constraint added.
- board-fit (LOW): `fan-master-preread`, `review-inline-code` and `twill-discussion-fork0-bench` fit; rejected as no record change needed, since D5 keeps the list from reviewers.
- leanness (MEDIUM): stencil-prose acceptance criteria conflict with commit e131c7625. Fixed: one criterion says review guards the wording.
- leanness (MEDIUM): measurement content is not Plan's. Fixed: moved to Turn 3.
- leanness (LOW): cut the tolerant-records, test-budget and stencil-ownership constraints, the duplicate docs constraint and the member-spelling assumption; shrink D3's path-existence rationale. Fixed.
- goal-scope (LOW): measurement baseline. Fixed in Turn 3.
- goal-scope (LOW): name members where read. Fixed in D2.
- goal-scope (LOW): D6 not asked for. Rejected: one small rule that keeps code names out of the record, in line with the operator's separate-file choice.
- goal-scope (LOW): Go changes licensed by implication. No action.

## Rejected alternatives

- A separate research agent between Discussion and Plan: the operator decided no new role; it adds a handoff that may lose understanding.
- Glyphs or code excerpts in the list (the Webster kick-start pack shape): the operator chose plain code names; excerpts grow the Discussion context, go stale, and pass the Step 2 bound.
- The list as `## Notes for the plan writer` in the decision record: the operator chose a separate file, and the record stays design only.
- An optional `relevant.md`: a writer could skip it with nothing noticing, and Plan would lose the head start silently.
- Checking that each named path exists: refuses legitimate lists naming a file the task deletes, or one a sibling landing renamed, for an advisory file.
- Adding the check inside `Validate`: darn and `decision add` would refuse runs with no list.
- Making a missing list a gate finding: in-flight runs and the fix-step gate, whose fixer never writes the list, would fail.
- Discussion-Review reviewing or fixing the list: the operator decided the review does not grow.
- Writing the list in darn: no Plan reads it.
- Adding a plan-lead peak column to tokencount: not asked for; the measurement can be read from the existing report and transcripts.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Required or optional output? Auto-pick: required, in the lead's `Outputs`, checked by `validate-discussion` (D1, D3).
- Q2 Format? Auto-pick: bullet per file, optional member names, ` — ` one-line why; no glyphs, signatures, `file:line` or excerpts (D2).
- Q3 How deep is the mechanical check? Auto-pick: existence by `Outputs`; the check covers at least one entry and each entry well-formed, tolerates absence, and does not check that paths exist (D3, revised after lens review).
- Q4 Does Plan tolerate a missing list? Auto-pick: yes, for runs whose Discussion predates the change (D4).
- Q5 Does any reviewer or the lens forks see the list? Auto-pick: no (D5).
- Q6 Darn? Auto-pick: out; no Plan step (Scope).
- Q7 Overlap with `## Notes for the plan writer`? Auto-pick: code list goes in `relevant.md` only (D6).
- Q8 Measurement? Auto-pick: no tokencount change; before figures from the board entry, after figures from tokencount over the next full loom runs (Interview Turn 3).
- Q9 Rework and prior-plan stencils? Auto-pick: unchanged; they replan from the decision record, prior plan and diff.
- Note: this run's own prompt asks for two files only, so this Discussion writes no `relevant.md` for itself.
