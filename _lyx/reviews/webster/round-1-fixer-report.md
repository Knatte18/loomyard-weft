# Webster-Review round 1 — fixer report

Every finding of `round-1-review.md` is fixed, each in its own commit on `refusal-audit` (not pushed).

## Fixed

- **F1 (MEDIUM)** — `09f9c441d`.
  In `ClassifyViolation`, the innermost worktree that holds a parent write now decides its class: a worktree enclosing this run's worktree is skipped for a write inside this one.
  A worktree nested inside this one still counts as another worktree.
  `TestClassifyViolation` gains a worktree nested inside the fixture's checkout, with three rows: from the outer geometry, a write into it is correctness; from its own geometry, an ignored write stays policy and a tracked write is correctness.
  The ignored row was confirmed to fail without the fix.
  This commit also rewrites the `ClassifyViolation` doc comment, which covers that site of F6.
- **F2 (MEDIUM)** — `05adc4211`.
  `failBatch` keeps the recorded blob for any path the batch already records, so a re-failed recovery checks the content the audit first flagged.
  `SuspectPath`'s doc comment says so, which covers the `state.go:184-185` site of F6.
  The new `TestPersistRecoveryTerminal_RefailKeepsFlaggedBlob` covers it.
  In that test, a recovery deletes the committed flagged file without committing the delete, and the blob survives the re-fail.
  A second recovery that restores and commits the flagged content is failed with the revert instruction.
  The test was confirmed to fail without the fix.
- **F3 (MEDIUM)** — `a89639c5b`.
  `Run` logs each `--fresh` drop warning through `logger.Warn` once the stale `state.json` is archived, so asking, died, timed-out and error outcomes still leave the drop on record.
  The done path keeps the `RunResult.Warnings` prepend.
  `TestRun_FreshDropsPathlessFinding`, whose Master ends asking, now asserts that the log names the dropped finding.
- **F4 (LOW)** — `621254a6b`.
  The `--card` usage error now ends in `way forward: re-run "lyx webster rebaseline" naming each changed card by its number, such as --card 5 or --card 05`.
  The `rebaseline bad card number` row in `contracts/specs/refusal-spec.md` quotes that clause, and `TestRebaselineCmd_RefusesNonNumericCard` asserts `way forward`.
- **F5 (LOW)** — `d35b3c376`.
  The docs and help text now describe the current behaviour:
  - `Rebaseline` says it restamps `PlanFileHashes` too;
  - `restampFingerprint`'s first line names both fields;
  - `persistPlanFingerprintRebaseline` says it persists the fingerprint and the per-file hashes;
  - `suspect.go`'s header covers accept-audit, `run --fresh` and recover-batch with each one's base;
  - the `run` command's `Long` no longer says `--fresh` acts only on a fingerprint mismatch.
- **F6 (LOW)** — `e5a48c72a`.
  The flagged comments in `doc.go`, `state.go` (`PlanFileHashes`), `gitwrap.go`, `audit.go`, `runlevel.go` and `recordbatch_test.go` are rewritten to one sentence per line, with breaks only at independent clauses.
  F1 and F2 had already rewritten the `ClassifyViolation` and `SuspectPath` comments.
- **F7 (NIT)** — `3605c6c0c`.
  The `rawGitMutationAllowlist` reason lists `status --porcelain` (dirty) and `status --porcelain --ignored` (ignoredPath) separately.
- **F8 (NIT)** — `1d556413b`.
  `fingerprint` reads its file set through `planFileNames`, the same function `planFileHashes` uses.
- **F9 (NIT)** — `a8a65bd2a`.
  `AcceptPendingAudit` collects each suspect path once.
  `TestAcceptPendingAudit_RefusesDifferingPath` now uses two findings that name the same file and asserts the refusal lists it once.

## Verification

The plan's full verify passes at `a8a65bd2a`: `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...`.
Each fix was also run green against its own tests before it was committed.

## Deferred

Nothing deferred.
