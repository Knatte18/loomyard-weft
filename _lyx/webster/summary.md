# Webster stability: gated turn ends, recovery writes, implementer commands, partitioned members

All three batches reported `done`, and no batch needed a recovery.

## Batch 01 — gated-turn-end-holds (cards 1 to 4, head `f416bcc3c`)

A gated run's waiting turn end no longer counts at once; the rule and its dead fields are dropped.
Webster's shell warning, note and summary section say the shell's result was never read.
A gate entry may supply its own one-line re-prompt renderer, and the verify entry's re-prompt and findings open with Master's fixer-fork way forward.
Deviations: `gate.go`, `spec.go` and `run.go` in `shuttleengine`, and `verifygate.go`, `verifygatereport.go`, `summary.go`, `triage.go` and `runlevel.go` in `websterengine`.

## Batch 05 — recovery-sessions-are-run-writes (cards 5 to 7, head `aee4414e1`)

Every recovery session is recorded on the batch and its writes count as the run's own.
Contract-file evidence counts recovery writes and names a fork or a recovery session.
A terminal stuck or dead recovery exits non-zero with the digest fields.
Deviations: `runlevel.go`, `archive.go`, `findingtext.go`, `runwrites.go` and `contractevidence_test.go` in `websterengine`, and `recoverbatch.go` and `acceptaudit.go` in `webstercli`.

## Batch 08 — implementer-allowed-commands (cards 8 to 13, head `53b12a693`)

The implementer body names the always-allowed commands and the tagged-test rule.
`internal/planglyph` gained a classifier that treats an ambiguous answer as partitioned over the enumerated build environments.
`statusFindings` and `createFindings` read it and render constraints, and done-check details name the lookup.
A partitioned Create or Rename-new lands; the delete-order, plan-gate and caller-coverage passes see a partitioned member's declarations; a partitioned Rename old side names its candidates and the way forward.
Deviations: `donecheck.go` and `handle.go` in `planglyph` each gained a small helper no card declared.

## Verify gate fixes

The gate's first attempt failed on `TestCLIForm_EveryCommand`: the `lyx webster recover-batch` `Long` help text was 2196 bytes against the 2000-byte cap.
The fixer fork tightened the wording in `internal/webstercli/recoverbatch.go`, keeping all the content, in commit `758935d6b` (`fix: shorten recover-batch Long help under the 2000-byte cap`).
It reports `lyx gate test --tags integration ./cmd/lyx ./internal/webstercli` and the comment lint passing, and the worktree clean.

## Digest notes

Every deviation is informational.
The forks reported all card gates and batch gates green.
