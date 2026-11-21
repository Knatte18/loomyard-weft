# Verify triage keeps first-run output, reports a command failure apart

The webster verify gate reruns a failing verify once; failures that pass on the rerun are recorded as flaky and reported as a warning, a `summary.md` section and the `webster-verify-triage` friction note.
Two faults made that report useless for diagnosis (issues #160 and #158).

## What changes

- `VerifyGateNotes` in `internal/websterengine` now records each flaky failure as the parsed `VerifyFailure` plus the first run's verify log path, not the identity alone.
  The friction note writes, under each identity, that log path and the failure's output tail (the tail `runplan.ParseVerifyRun` already capped at 40 lines), as a fenced block, or says the output was empty.
- A first run whose output named no failing test or package parses to the opaque `verify-command` identity.
  It is no longer listed as a flaky test:
  the warning, the friction note and the `## Integration suite triage` section of `summary.md` each report it as a failure of the verify command that passed on rerun, named by its first run's log.
- `AppendIntegrationTriage` takes the verify command failures' logs as a second list.
- The package doc and `contracts/specs/webster-spec.md` say what the note keeps and how a verify command failure is reported.

## Design decisions

- The parser keeps `verify-command` as the opaque identity; only the triage's reporting changes.
  The verify-gate report and the fixer routing already treat an opaque failure apart, so the fault was confined to the flaky path.
- No filter for the "Shell cwd was reset to …" line.
  No lym code writes it, and the verify log holds only the verify command's own output;
  it is the agent harness's notice, printed when an agent read the note.
  Filtering an engine's notice by its text would hard-code one engine's wording.
- No change to `internal/reedcli`.
  `lym gate test --gate webster ./internal/reedcli` passes on this base.
  The failure #158 reports is in that run's verify log (`single-repo-fabric-mode`, 2026-11-19 09:57Z):
  reedcli "failed outside any test" on tmuxkit's stray `/tmp/.git` check, which then failed every package running when the entry appeared.
  Commit `5d8724d0` in this base already makes such a sighting a machine finding unless the package's own process worked in that directory;
  no package in the gate run logs written since that commit is failed for it.
  The process that creates `/tmp/.git` is still unidentified; with this change, the next such flaky failure carries the tmuxkit line in its note.

- The red-first check is waived for this change, by the parent's recorded "No test seam" decision.
  The diff updates `contracts/specs/webster-spec.md`, a non-Go file the check cannot put back on the merge base.
  `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` reaches the bug through code the base has (`newVerifyGate` and `VerifyGateNotes.Apply`).

## Check items

- The friction note's wording and layout, in `TestWriteTriageFrictionNote`.
- `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` drives the gate through a failed first run and a passing rerun, then `VerifyGateNotes.Apply`, for a failing package and for a verify command that named no test.

## Merge danger

**Door:** two-way
**Blast radius:** webster
