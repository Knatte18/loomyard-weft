# Fixer report — discussion round 3

I checked all three findings against the code and fixed them.
I also added the operator's #518 scope change, relayed by `ly:orch`, to D2.

## Fixed

- F1 (BLOCKING, design): D2 now counts an unquoted `#` anywhere in a word as a glob character, under the same rule as `*`, `?`, `[` and a leading `^`.
  `plainWord` (`internal/fabricengine/refscanner.go:264`) admits `#` today, so the premise holds.
  New rows: `git log x#--output=f` is rejected and `git log --oneline -- x#` is admitted.
  The rejected alternative in the support log about narrowing the bound now names both `^` and `#`.
- F2 (MEDIUM, design): I dropped D6's dead-fork case and its three dead-fork rows.
  The premise holds: `DigestStatusDead` is set only by `Classify` (`classify.go:44-52`), which only recover-batch's wait calls (`recoverbatch.go:728`, on the recovery record).
  `distill` (`digest.go:44-52`) and `failBatch` (`batchfail.go:136`) yield done, stuck or failed.
  D6 now states that only a recovery record is ever terminal dead, and the spec rewording covers four cases.
  To keep the count-against-amendment case pinned on a reachable state, I added a row for a failed fork record at the cap with an unrendered amendment, which names `recover-batch NN`.
  Support log item 7 and the ledger are corrected.
- F3 (MEDIUM, decision): D6's exhausted case now wraps `ErrRecoveryExhausted`, and the begin-batch verb sets `recovery_exhausted: true` on it, as `internal/webstercli/recoverbatch.go:196-197` does.
  Master's begin-batch existing-report rung gains one sentence: that flag ends the run stuck, and the reset steps are left to the operator.
  Scope now lists the begin-batch verb and that rung sentence, and Scope Out excludes every other rung.
  The Merriam-consistency sentence covers the exhausted case at every status, and a bound states what the flag can skip.
  There is a new acceptance row for the envelope flag, plus a docs line for the `doc.go` recovery section and the rung.

## Scope addition: #518 (from `ly:orch`)

D2 drops the exact unquoted words `2>&1` and `2>/dev/null` before the whole-command `>` and `&` scans.
It also admits `sed` in one form: `-n`, a line-print script, and plain file words that don't start with `-` and contain no unquoted glob.
The orch's message named only head, tail, grep and cut as readers.
I added `sed` because #518's own example opens with `sed -n 285,300p`, and that example would still halt without it.
The `sed` form is bounded against `-i`, `w` and `e` scripts, and against GNU's practice of reading flags placed after file operands.
Rows: the #518 compound and a help call with `2>&1` piped into `head` are admitted.
Other redirections, `sed` writing forms and a loop with a redirection are rejected.
The support log records the question, the auto-pick and two rejected alternatives.

## Disputed

None.

## Deferred

None.
