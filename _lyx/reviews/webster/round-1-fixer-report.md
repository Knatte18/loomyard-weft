# Webster-Review round 1 — fixer report

All 11 findings were fixed, one commit each.
After the last fix, `go build ./...` and the plan's `verify:` test commands (untagged and `-tags integration`, `-count=1`) all pass.

## Fixed

- **F1** (`2b5cdb05e`): the new `gitwrap.go` comments now use semantic line breaks.
  This covers the header lines and the `refuseMidMerge` and `reconcileReportHead` docs.
- **F2** (`6953eb62d`): rewrote the `doc.go` merge-only paragraph with semantic line breaks.
  It now states once that both verbs apply one rule, and gives the acceptance rule its own sentence rather than hanging it off a recover-batch clause.
- **F3** (`3f859d871`): re-broke the changed comments in `recordbatch.go` and `recoverbatch.go`.
  These are the `RecordResult`, `postBatchInputs` and `RecoverResult` docs, the postBatchChecks delta note, and the `PersistRecoveryTerminal` note.
- **F4** (`195b0014a`): the `RunInFlight` doc now breaks at its semicolon clause.
- **F5** (`fea90e5fd`): the `websterInFlightWarnings` doc breaks at its semicolon, and the `addMergeVerbs` `loc` sentence is on one line.
- **F6** (`3fdc8ae0c`): the merge-in warning and the `merge-in` `Long` help now say that record-batch tolerates a parent merge commit after a fork's commit but refuses a fast-forward past it.
  The warning is therefore true for a fast-forwarded merge-in as well.
- **F7** (`5f933726c`): the new test comments use semantic line breaks.
  The files are gitwrap_test, recordbatch_test, recoverbatch_test, parentmerge_test and mergein_webster_integration_test.
- **F8** (`48ac093a4`): `assertOneWebsterWarning` now checks the warning's content (Webster mid-run, `lyx webster record-batch`, pre-merge trees).
  The helper covers both the clean and the conflict envelope.
- **F9** (`03bdbed21`): the non-merge-between-merges refusal now asserts the phrase, both SHAs and the merge-only rule.
  The report-head-is-merge case asserts the exact walked list `(merge2)`.
- **F10** (`6e6934f85`): the mid-merge refusal is now prefixed `webster:`, and the wrapped probe failure keeps `websterengine:`.
- **F11** (`a3028506c`): dropped the `newHead` alias.
  The test now asserts that the merge appears both as the new HEAD and as the one walked merge.

## Deferred

None.

`gofmt -l` also flags alignment in `recordbatch_test.go` and `strand_test.go` on the fake-engine one-line methods.
Both predate this branch's merge base and are outside the review range, so they are not findings and were left untouched.
