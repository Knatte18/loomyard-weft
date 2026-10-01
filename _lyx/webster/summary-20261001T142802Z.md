# Refusal audit: every webster, shed and loom refusal leaves a way forward

All 43 cards are done and committed; no batch failed or needed recovery.
The plan-level integration suite (`go build`, `go test`, integration-tagged tests, `go vet -tags smoke`) reported `status: OK` at HEAD `5aacd53984fffab5e0bcd467883406de681ef835`.

This session completed the cards added late in the plan:

- **43 refusal-spec-rework:** the refusal table, `docs/overview.md` and the ly-drive skill follow cards 32–42. It ran before its dependencies were in the tree, so its cells were not checked against the final messages in code.
- **36 fabric-reference-mutation-class:** `AuditViolation` carries the command; a fabric reference whose command can change files is classed correctness, others policy.
- **37 rebaseline-card-hashes:** begin-batch and recover-batch record each card's SHA-256, and rebaseline refuses a begun card whose content changed.
- **38 run-exit-pending-findings:** run-exit correctness findings stay pending in state and block run entry (`ErrPendingAuditFindings`) until accepted.
- **33 goto-admissible-targets:** `Goto` refuses a running run and any target past the reference row.
- **34 goto-way-forward-address:** goto way-forward texts carry the run-id and the recipe's own missing-status clause (loom and batten fill them).
- **35 batten-goto-teardown:** a `PreGoto` hook lets batten admit Worktree-Teardown only once the child run is done.
- **32 loom-preflight-goto-reentry:** Loom-Preflight passes a deliberate goto re-entry and names `lyx loom goto --to Loom-Preflight` otherwise.
- **39 accept-audit-verb:** `lyx webster accept-audit` accepts the pending run-exit findings.
- **40 loom-approve-way-forward:** approve's refusals name `git push`/`git pull` and a backward goto to Publish.
- **41 master-stencil-done-checks:** the Master stencil no longer lists later-card drift as a batch_failed done-check.
- **42 fabric-words-outside-fabric:** the fabric side names added in tests were reworded.

Deviations: batch 37 also changed `internal/websterengine/beginbatch_test.go` and `recoverbatch_test.go`, outside its declared file-ops, because the shared fixtures now need real card files.
The worktree carries a pre-existing uncommitted change to `internal/shedengine/historyfold_test.go` that no fork committed.
