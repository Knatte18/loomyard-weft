# accept-audit can never accept a fork-contract-write finding

`webster-accept-audit-contract` · Note · bug, webster

Imported from [issue #363](https://github.com/Knatte18/loomyard/issues/363).

Seen 2026-10-04 in `board-model`: the run-exit audit flagged the verify-gate fixer fork for writing `_lyx/webster/outcome.yaml` (and `summary.md`), Master's contract files; Master had rewritten both afterwards.
`lyx webster accept-audit` refused even after HEAD was reset to the last batch head and both untracked files were deleted (their state at that head): "cannot be checked: …/_lyx/webster/outcome.yaml, …/summary.md; way forward: reset the branch to the run's start commit … and run lyx webster run --fresh".
`checkSuspectPaths` treats weft paths as unverifiable, so a `fork-contract-write` finding can never be accepted; the offered way forward throws away every batch (10 here).
The hub cleared `pendingAuditFindings` in `state.json` by hand to continue.

Expected: a contract-file finding is acceptable once the contract files are absent (Master rewrites them at its next terminal turn), or the gate fixer is kept from writing them in the first place.
Related: #361 (the same run's gate hang).


## From issue #365

Imported from [issue #365](https://github.com/Knatte18/loomyard/issues/365).

`lyx webster accept-audit` refused to accept pending audit findings:

```
webster: pending audit findings cannot be accepted: cannot be checked: _lyx/webster/outcome.yaml, _lyx/webster/summary.md; way forward: reset the branch to the run's start commit <sha> with git and run "lyx webster run --fresh"
```

Problems:

- The refusal says the two files "cannot be checked" but not why (missing, unreadable, changed since the audit, or not matching an expected state), so the caller cannot tell what to repair.
- The only way forward offered is resetting the branch to the run's start commit and rerunning with `--fresh`, which discards the run's work.
  There is no lighter path, such as regenerating the two files or re-running the audit.

Provenance: task `board-model`, note `webster-refusal-accept-audit.md`.
