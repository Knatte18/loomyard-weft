# Burler fixer-report — code round 1

## Changes

All six findings fixed as proposed; the gate on `./internal/loomcli` and `./internal/discussionparser` is green.
Commits group findings by package rather than one per finding, because F1, F2, F3 and F5 share `decision.go` and `decision_test.go`.

- F1: `decisionEntries` refuses an unknown key with `unknown field "<key>"` and the way forward, through `unknownEntryKey`, a second pass over the file's keys; a "from entry with an unknown key" row in `TestDecisionVerb_Refusals`.
  The proposed `DisallowUnknownFields()` was not used (fixed differently): `TestTolerantRecords_NoStrictJSONDecode` (`PATTERN-tolerant-records`) refuses any call to it, as the tier-1 gate showed at `internal/loomcli/decision.go:149`.
- F2: the `decoder.More()` test is replaced by a second decode into `json.RawMessage` that must return `io.EOF`; a "stray closing bracket after the array" row.
- F3: the fake gained `amendErrTitle`, and the amend-plan table gained a `from` field and a "BatchSecondTitleFails_RestoresTheAmendmentRecord" row.
- F4: `internal/discussionparser/doc.go` names `AppendDecisions`, placed after the PATTERN-change sentence so "That change" keeps its antecedent.
- F5: the add help says the per-entry flags are required unless `--from` is passed.
- F6: the `AppendDecisions` doc comment states its behaviour without reference to `AppendDecision`.

Self-check changes: the `decisionEntries` doc comment lists the new refusals, and the help paragraph is rewrapped.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `decisionEntries` doc comment omits the new refusals; fixed as proposed.
- fix-consistency, LOW: `append.go` header and `AppendDecision` doc stale after the batch append; disputed, premise false (see Disputed).
- fix-proposal, LOW: the inserted `doc.go` sentence broke the antecedent of "That change"; fixed as proposed.
- fix-proposal, LOW: ragged line in the add help; fixed as proposed.

## Disputed

- fix-consistency second finding, ground `premise false`: the `append.go` header at lines 1-5 already says "AppendDecisions adds several such calls in one write", and `AppendDecision` is still accurately described as the one-call wrapper.
  The finding was not part of F6.

## Deferred

None.
