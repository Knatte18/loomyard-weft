# Fixer report — discussion round 1

All five findings fixed; `lyx loom validate-discussion` exits 0 on the result.

## Fixed

- **F1 (BLOCKING), launch-script lifecycle.**
  `decision-record.md`: the cleanup decision now states the rule as "a launch script exists only while its GUID is in reed's persisted strand table", and derives the deletion points from it — the shared removal step behind `RemoveStrand` and `ReplaceStrand`, `Down` deleting the launch directory with `reed.json`, and a failed add deleting its unpersisted script.
  The false "`Down` keeps the scripts for `Resume`" premise is gone.
  Scope In and the cleanup acceptance criterion match the rule.
  `support-log.md`: Interview turn 7 and ledger Q7 record the revision.
- **F2 (MEDIUM), Windows ExecutionPolicy.**
  `decision-record.md`: pwsh's typed line is now `. ([scriptblock]::Create(<ReadFile(path)>))`, which no ExecutionPolicy governs, with the never-fail-a-launch rationale; POSIX keeps `. '<path>'`.
  The ExecutionPolicy risk and the `RemoteSigned` Auto-mode assumption are removed, and the `internal/shell` acceptance criterion pins the pwsh form.
  `support-log.md`: the file-path dot-source moved to Rejected alternatives with the policy reason, the `Invoke-Expression` entry now says why the script-block form is preferred, and Interview turn 4 and ledger Q8 record the revision.
- **F3 (LOW), Q10.**
  `decision-record.md`: the scrollback-reader risk became an acceptance criterion requiring any test or tool that matches the launch line to be retargeted.
  `support-log.md`: Q10 resolved with the production-reader reasoning.
- **F4 (LOW), equivalence overclaim.**
  `decision-record.md`: the Goal and the script-content rationale now claim what holds — same statements in the pane shell's own scope, so exports, `PATH`, working directory and exit status match — and name the interactive-only processing that no longer applies.
- **F5 (NIT), leanness.**
  `decision-record.md`: removed the per-strand-directory clause from the location rationale, the relaunch-overwrite risk, and the two Auto-mode assumptions restating Scope Out and the location decision.
  Open risks now reads "None beyond today's behaviour."
  `support-log.md`: the per-launch-filenames rejected alternative carries the reason no overwrite race exists.

## Deferred

None.
