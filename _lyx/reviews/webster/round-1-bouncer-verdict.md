---
verdict: APPROVED
rationale: "All eleven round-1 findings (F1-F11) are fixed in commits 7560f391b..84d95da0e, each verified against the code; the fixes add no new defect, and go build, go vet (no tags, integration, smoke) and go test -tags integration over websterengine, webstercli and cmd/lyx pass."
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`, from `product.parent: main` in `_lyx/shed/refusal-audit/status.json`.
The round-1 report carries no `## Focus departures` section, so there is no departure to ratify or reject.
Its eleven findings stand as real defects against cards 55–64 and `ca97012fa`, and I accept its "settled as not a defect" list for cards 56–60, 63 and the Edit-group blast radius.

## Fix verification

- **F1** `7560f391b`: `recoverSpawn` and a re-begin in `BeginBatch` carry `prior.ForkTranscripts`, so `forkOwnReport` maps a recovered batch's original fork to its own report.
  No reader of `ForkTranscripts` keys on `Kind == "fork"`, so carrying it onto a recovery record changes nothing else.
- **F2** `c0e7ce67a`: `auditTerminalFork` returns nil, nil while another fork batch of the same `SessionID` is begun and not terminal, or the integration report exists, leaving the already-terminal refusal and consuming no transcript.
  A batch begun in an earlier Master session does not trip the skip, which is right, since its fork's transcripts are not in this session's audit.
- **F3** `eb158b4e9`: `evidenceBases.Starts` lets `freshPendingDrop` tell "nothing recorded" from "none qualifies".
  `headBeforeEveryStart` refuses unless HEAD is an ancestor of every recorded start, and names `git merge-base --octopus`.
  The new refusal-spec row quotes that text.
- **F4** `6be1c707e`: `pendingPathsWayForward` sends a pathless finding, or a non-plan path `trackedRel` rejects, to the `--fresh` route.
  That matches `AcceptPendingAudit`, which refuses any finding set holding an unverifiable path or a pathless finding.
  The four refusal-spec cells and the unchanged fork-state-write row now quote what the code emits.
- **F5** `322dd78a5`: `reconcileHead` takes a `headRefusal`.
  accept-audit's text names "the last batch head" and `re-run "lyx webster accept-audit"`, and record-batch and recover-batch keep theirs.
  Nothing parses the old `remedy:` label.
- **F6** `55f388cc3`: the `trackedRel` gate is gone, and an empty `StartSHA` is treated as holding nothing rather than reading the index.
  `BeginBatch` and `recoverSpawn` always record a start, so the empty-start branch only guards a legacy state.
- **F7** `f7e5c4062`: `gitwrap.go`'s header names `treePathsWithBlob`, `shaExists` and `isAncestor`, and the guard's allowlist reason names `ls-tree -r` and every `gitrepo.New` query.
- **F8** `2a8b7253f`: `storePlanBaseline` re-hashes the bytes it read and refuses on a mismatch.
  In `Run`'s `--fresh` branch the refusal lands after the archive, but a re-run then takes the `st == nil` branch and the drop warnings were already logged, so the "transient, re-run" way forward holds.
- **F9** `0045d876d`: every listed comment breaks at its clause boundary.
  A scan of every comment and Markdown line added since `ca97012fa` finds no unbroken semicolon or independent ", and/but/or" clause.
- **F10** `d244cf499`: `findingsEnvelope` takes the message, and both validate branches call it.
- **F11** `84d95da0e`: the recovery plan-path way forward uses `%02d`.
  The fixer's correction is accurate: some older recover-batch messages still use `%d`.
  That inconsistency predates cards 55–64 and was not part of F11's subject.

## Per-card mechanical check

No card in the plan has a `Delete`, `Rename` or `Move` group, so neither `assert-no-callers` nor the AST-plus-grep check applies.
The Edit groups' changed signatures, including the fix range's new `pendingPathsWayForward(…, pathless, …)`, `reconcileHead` and `findingsEnvelope(out, msg, …)`, compile and vet under no tags, `integration` and `smoke`.
