# Round 2 fixer-report — discussion

## Changes

- F1, MEDIUM, fixed as proposed, with one adjacent edit: D2 adds "Its prose gains the emphasis line after its list, …" after "`way-forward` stays defined and stays in `twill-plan`", and D2's acceptance line adds "and its prose ends with the emphasis line".
  The next sentence's subject, "Its prose owns the repository invariants", became "`repo-invariants`' prose owns …", since the inserted sentence would otherwise make "Its" read as `way-forward`.
  The self-check then narrowed the proposed "as the other `twill-plan` lenses end" to "as `coverage`, `row-validity` and `forced-edits` end", since `guard-reach` is in `twill-plan` and stays a procedure lens ending in its list (`internal/lenses/template.yaml:280-287`).
- F2, LOW, fixed as proposed: D5's third Bound bullet names pointing the code review at any other defined fan beside solo, and the whole-diff read under the code-review rubric as the floor.
- F3, LOW, fixed as proposed: Auto-mode assumptions gain "D1: `review-discussion` drops `goal-scope` and `board-fit` with no replacement, which the entry does not name."
- F4, NIT, fixed differently: D5 names `PrepareRunSettings` and a reconcile of the local file as the routes where `MigrateConfig` runs, and says a hand-edited local file holding a webster key is refused at load until a reconcile migrates it.
  The proposed tail "as for every retired key" was not kept: a declared retired key of category none is admitted at load (`internal/configengine/layers.go:275-280`, loom's `RetiredKeys` `fan_review` at `internal/loomengine/config.go:555-557`).
  The sentence instead says the webster keys are refused as keys outside the loom template, since neither is declared one of its retired keys (`layers.go:283-284`).
- Support log: a `### Round 2` entry under Review rounds lists the four fixes, matching round 1's entry.

## Self-check

- fix-consistency, MEDIUM: "as the other `twill-plan` lenses end" is false for `guard-reach`. Fixed as proposed, merged with fix-claims finding 1.
- fix-consistency, LOW: the support log has no round-2 entry under Review rounds. Fixed as proposed.
- fix-claims, MEDIUM: the same `guard-reach` defect as fix-consistency's MEDIUM finding. Merged; fixed as proposed.
- fix-claims, MEDIUM: "as for every retired key" does not hold, because a declared retired key of category none is admitted. Fixed with own change: rewritten to say the webster keys are refused as keys outside the template, not declared retired. Merged with fix-proposal's finding.
- fix-proposal, MEDIUM: the same retired-key defect as fix-claims finding 2. Merged; fixed as above.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
