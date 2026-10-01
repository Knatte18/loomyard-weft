# Webster-Burler round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit each.
After the last commit, the plan's verify commands all pass: `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...`.

## Fixed

- **F1 (BLOCKING)** `7560f391b`: `recoverSpawn` and a re-begin in `BeginBatch` now carry `prior.ForkTranscripts` onto the fresh record, as they already do for `AuditWarnings`.
  The run-exit audit then maps a recovered batch's original fork to its own report.
  Tests: `TestRecoverSpawn_CarriesSuspectPathsAndTranscripts` (renamed from `TestRecoverSpawn_CarriesSuspectPaths`) and `TestBeginBatch_ReBeginKeepsAuditWarnings` assert the transcripts are carried.
- **F2 (MEDIUM)** `c0e7ce67a`: `auditTerminalFork` returns nil, nil, leaving the already-terminal refusal, while another fork batch of the same session is begun and not terminal, or while the integration report exists.
  `doc.go` states the rule.
  Test: `TestRecordBatch_TerminalAuditSkipsAnotherForksTranscript` covers both cases, with the done record unchanged and the other fork's transcript left unseen.
- **F3 (MEDIUM)** `eb158b4e9`: `evidenceBases` gains `Starts`.
  When starts are recorded but none qualifies, `freshPendingDrop` calls the new `headBeforeEveryStart`, which refuses unless HEAD is an ancestor of every recorded start and names `git merge-base --octopus <starts>`.
  `refusal-spec.md` gains a `fresh: recorded starts diverge` row.
  Test: `TestRun_FreshDivergentStartsNeedHeadBeforeEvery` refuses with HEAD on one diverging line, then re-initialises once HEAD is reset to the common root.
- **F4 (MEDIUM)** `6be1c707e`: `pendingPathsWayForward` takes `pathless`.
  When a finding has no path, or a non-plan path is outside the tracked tree (`trackedRel`), it returns the `--fresh` route as the whole way forward, since accept-audit would refuse the rest along with it.
  `pathlessClause` is gone.
  The four refusal-spec cells that quote the run-exit way forward now say "for a finding with no path or a path nothing the run recorded can check".
  Test: `TestRun_ForkStateWriteAtRunExit` asserts the `state.json` row's stuck reason names `lyx webster run --fresh` and not accept-audit.
- **F5 (LOW)** `322dd78a5`: `reconcileReportHead` delegates to the new `reconcileHead`, which words its refusals through a `headRefusal`.
  record-batch and recover-batch keep their text, but the unqualified-merge refusal now says `way forward:` instead of `remedy:`.
  accept-audit passes `acceptAuditHeadRefusal`, so its message names "the last batch head" and `re-run "lyx webster accept-audit"`.
  The refusal-spec row now quotes that text.
  Tests: `TestAcceptPendingAudit_RefusesCommitPastHead` asserts the new way forward, and the three tests that asserted `remedy:` now assert `way forward: move HEAD back`.
- **F6 (LOW)** `55f388cc3`: `checkRecoveredSuspects` drops the `trackedRel` gate on the blob search, and treats an empty `StartSHA` as holding nothing instead of calling `commitBlob`, which would read the index.
  Tests: `TestPersistRecoveryTerminal_FailsWhenSuspectUntrackedAndMoved` and `TestCheckRecoveredSuspects_EmptyStartHoldsNothing`.
- **F7 (LOW)** `f7e5c4062`: `gitwrap.go`'s header names `treePathsWithBlob`, `shaExists` and `isAncestor`.
  The raw-git guard's allowlist reason names `ls-tree -r` and every read-only `gitrepo.New` query the file makes, including the pre-existing `MergeHeadPresent` and reconcile-walk queries it also omitted.
- **F8 (LOW)** `2a8b7253f`: `storePlanBaseline` hashes the bytes it read and refuses, storing nothing, when they no longer match the recorded hash.
  Test: `TestStorePlanBaseline_RefusesBytesNotMatchingTheirHash`.
- **F9 (LOW)** `0045d876d`: every listed comment and the Master stencil line now break at the clause boundary, and `validate.go`'s fixed-column comment is rewrapped one sentence per line.
  The new comments written for F2 and F3 follow the same rule.
- **F10 (NIT)** `d244cf499`: `findingsEnvelope` takes the error message, and validate's plan-edit branch calls it instead of building the JSON by hand.
- **F11 (NIT)** `84d95da0e`: the recovery plan-path way forward formats the batch with `%02d`.
  One correction to the review: its summary says every other recover-batch way forward uses `%02d`.
  Several older recover-batch messages (the malformed-report and transient ones) use `%d`, so the fix aligns this one with `RecordBatch`'s terminal refusal, `RecoverSpawnOrAttach` and the card's `NN`, not with every message.

## Deferred

None.
