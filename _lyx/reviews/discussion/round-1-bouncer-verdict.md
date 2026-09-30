---
verdict: APPROVED
rationale: "All five round-1 findings are resolved in the current pair: the lifecycle rule is keyed to the persisted strand table and covers RemoveStrand, ReplaceStrand, Down and a failed add; pwsh sources a script block over ReadFile; Q10 is an acceptance criterion; the equivalence rationale is scoped; and the leanness cuts landed without losing a requirement."
---

The round-1 report raised five findings against an earlier revision of the pair.
I checked each against the current `decision-record.md` and `support-log.md`, and each is resolved.

- **F1 (BLOCKING), launch-script lifecycle.**
  The Decisions entry now states the rule in terms of the strand record: a script exists only while its GUID is in reed's persisted strand table.
  Scope In, that decision and the cleanup acceptance criterion all name the shared removal step behind `RemoveStrand` and `ReplaceStrand`, plus `Down` deleting the launch directory with `reed.json`, plus a failed add.
  The false premise that `Down` keeps strands for `Resume` is gone.
  A spot-check of `internal/reedengine/lifecycle.go` confirms that `Down` deletes `reed.json`, and `removeStrandLocked` in `strand.go` is the shared step.
- **F2 (MEDIUM), ExecutionPolicy.**
  pwsh's `Source` is now `. ([scriptblock]::Create(<ReadFile(path)>))`.
  `internal/shell/pwsh.go`'s `ReadFile` is the `(Get-Content -Raw …)` idiom, so the form is sound and no ExecutionPolicy governs it.
  The risk and the RemoteSigned assumption were dropped, the acceptance criterion forbids a file-path dot-source, and the rejected alternative sits in the support log with its reason.
- **F3 (LOW), Q10.**
  Q10 is now an acceptance criterion: any test that matched pane text against the launch line gets retargeted.
  Q10 in the support log is resolved with the reasoning about the production readers.
- **F4 (LOW), overclaim.**
  The rationale for the script content now claims only what holds: the same statements run in the pane's own scope, and interactive-only processing no longer applies.
- **F5 (NIT), leanness.**
  The location rationale no longer carries the per-strand-directory clause.
  The relaunch-overwrite risk is deleted, and its reasoning moved to the per-launch-filenames rejected alternative, with the atomic-write requirement still in Decisions.
  The two restating Auto-mode assumptions are gone.

The report has no `## Focus departures` section, so there are no departures to ratify.

A fresh read against the rubric turns up no new finding.
The `## Review rounds` section still reads `_No rounds yet._`, but the stencil makes that section the gate's to append, so it is not a writer deficiency.
The optional Notes subsection and the lack of call-site enumeration fall under the Do-not-flag list.
