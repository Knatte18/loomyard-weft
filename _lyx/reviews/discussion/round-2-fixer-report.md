# Round 2 fixer report

All nine findings are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` needed no change.
`lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1** D13 now says the shared rule is the audit's own patterns (separator-prefixed name, bare word outside quotes, spelling).
  The audit's exact-path branch matches nothing the separator-prefixed name does not, so the audit's matches stay unchanged.
  The patterns are declared once on the fabric side, and `planparser` receives a matcher through a narrow seam from the plan-format validation's callers, importing neither the fabric side nor `weftname`.
  The Constraints bullet and the Out of scope audit line say the same.
- **F2** D14's reader list is now closed and named: `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq`, `cut`.
  `sort` and `uniq` are dropped, with the reason, and any unnamed command makes the entry not read-only.
  The exclusion list shrank to what the closed list does not already cover (redirection, substitution, backgrounding).
  The acceptance criterion and the Open risks bullet follow.
- **F3** D14's bound now states what the evidence shows (start in HEAD's history, nothing uncommitted, no listed reader can write) and that it does not inspect the batch's commits.
- **F4** D11 now attributes per-batch figures to each batch's own profile, so a multi-profile run contributes to each; its findings per card appear in its own row only.
  The acceptance criterion states the same.
- **F5** D11 now keeps a `--fresh` run out of every profile line and lists it in a marked row, since its figures cannot be paired across attempts.
  The acceptance criterion states the same.
- **F6** D11 names the run's plan commit as the card-count source for a run with no recorded partition.
- **F7** D13 now gives the reason the span rule differs for the spelling and the path.
- **F8** The Goal now names the refused and accepted classes the Decisions define, and D5 says "multi-card batch".
- **F9** D4 now points the operator at the batch's recorded `Estimate` and position in state.json instead of the calibration report.

## Deferred

None.
