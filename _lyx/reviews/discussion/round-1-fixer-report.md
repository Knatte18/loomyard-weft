# Fixer report — discussion round 1

Every finding was validated against the code and fixed; `lyx loom validate-discussion` exits 0 after the edits.

## Fixed

- **F1** (BLOCKING): confirmed against `beginbatch.go` (a re-begun fork record carries `prior.Recoveries`) and `RecoverSpawnOrAttach` (refuses at `maxRecoveries` unless `rendersNewAmendment`).
  D6 now cases the dead record in recover-batch's order: at the cap with no unrendered amendment, either kind, is exhausted with both reset routes; then a dead fork record names `recover-batch NN`; then a dead recovery earning `RecoveryRetry`, or at the cap with an unrendered amendment, names it once more; otherwise the run ends stuck.
  The rationale states why kind cannot be keyed before count.
  Acceptance gains rows for a dead fork record at the cap and a dead recovery at the cap with an unrendered amendment; the plan-writer note names the amendment input.
- **F2** (MEDIUM): D2 now rejects a glob character in a `lyx` word, quoted or not, so `lyx fabric diff '*'` stays rejected as Acceptance says.
- **F3** (MEDIUM): D3's pin moves to `cmd/lyx` against `newRoot()` and covers local and persistent `--json`; Scope In names `cmd/lyx`; Acceptance gains the pin row; the gate line runs `./cmd/lyx`.
  The `internal/clihelp` fallback is gone from Open risks and the question ledger, and the support log records it as a rejected alternative.
- **F4** (MEDIUM): D2 rejects an unquoted parenthesis and states the bash-or-zsh assumption, naming zsh glob qualifiers and bash extglob in the bound; Acceptance rejects `git log -- *(.)` and `git log -- @(a|b)`.
- **F5** (LOW): unquoted globs after `--` are admitted only in `log`, `show`, `diff` and `status`; the rationale is narrowed to those forms and the bound names `ls-remote` and `remote` as rejected; Acceptance rejects `git ls-remote -- *` and `git remote show -n -- *`.
- **F6** (LOW): confirmed at `internal/webstercli/cli.go:298` and `rebaseline.go:98`; Scope In adds `internal/webstercli`, and the gate line runs `./internal/webstercli`.
- **F7** (LOW): the two Auto-mode lines restating D1 and D5 are deleted; D1's second sentence and D5's rationale keep both requirements.
- **F8** (NIT): the two Open-risks lines are deleted; neither carries a requirement.
- **F9** (NIT): confirmed at `internal/fabricengine/refscanner_test.go:197`; D4 marks the `log` `--output` row as already present.

## Disputed

None.

## Deferred

None.
