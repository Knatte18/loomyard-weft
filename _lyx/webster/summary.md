# plan-card-rules: five batches, 27 cards

All five batches ended `done`.
Batches 01 and 23 first failed webster's done-check and were each finished by one `recover-batch` run.
The done-check reported plan targets of the form `plan:<unit>#Test…` as unresolved because the tests are declared in the external test package (`<unit>_test#Test…`).

## Batches

- **01-plandag-additive-guess** (cards 1 to 6): `plandag -additive-guess`, the pattern overview projection, planparser's entry lines, the three plan-gate pattern checks, the merge-base overview wiring and the `Extends` label parse.
  `record-batch` failed on three unresolved test targets; `recover-batch 01` finished the batch at `95b37430900f53dd4d7a2032acf95392c943f28e`.
  Deviations: `internal/loomcli/parity_pattern_integration_test.go`, `internal/loomshed/fixture_test.go`, `internal/loomshed/patternplan.go`, `internal/pattern/projection.go`, `internal/planparser/extends.go`, `internal/planparser/parse.go`, `tools/plandag/render.go`.
- **07-extends-group-readers** (cards 7 to 12): `Extends` group readers, the shared pair rule, the plandag `E` mark, the plan-review guidance, the amendment record and the settle round start.
  Recorded `done` at `b5e2f7da6f6137c7cc668fa539ce1e50a77d9b69`.
  Deviations: `internal/loomengine/config.go`, `internal/planparser/pairrule.go`, `internal/planparser/pairrule_test.go`, `internal/websterengine/sequence_test.go`, `tools/plandag/graph.go`.
- **13-decision-amends-plan** (cards 13 to 17): `decision add --amends-plan`, the settled-guard admission, the commit-seam and rework handling of the amendment record, and the Plan-Burler-Review instruction.
  Recorded `done` at `a7d70108f036ce7919b5ea366c7d02dfbbbf1b6f`.
  Deviations: `internal/loomcli/decision.go`, `internal/loomcli/decision_test.go`, `internal/loomcli/settledguard.go`, `internal/loomcli/wiring.go`, `internal/loomshed/generation_test.go`, `internal/loomshed/prrework_test.go`.
- **18-recipe-amendment-target** (cards 18 to 22): the `amendment_target` recipe key, the arbiter and orch stencil routing, Webster's plan-conflict way forward and the Publish failure record's trace fields.
  Recorded `done` at `3df6077f1decfb080623fa6b21acb714c895ee52`.
  Deviation: `internal/loomrecipe/names.go`.
- **23-planglyph-enclose** (cards 23 to 27): the `EncloseAt` wrapper, planparser's target-holds decision, the `failuretrace` package, the landing trace seam and the Publish failure note.
  `record-batch` failed on three unresolved test targets; `recover-batch 23` finished the batch at `507fd63bc66179516d09e2e73eaddf3cf4d434fa`.
  Deviations: `internal/failuretrace/locations.go`, `internal/failuretrace/locations_test.go`, `internal/failuretrace/trace.go`, `internal/failuretrace/trace_test.go`, `internal/landingshed/mergein_test.go`, `internal/loomshed/publishfailure.go`, `internal/loomshed/publishfailure_test.go`, `internal/planparser/targetdirs.go`.

## Scope guard and departures

The scope guard listed the deviations above for each batch; all are informational.
No batch digest carried a `Pattern-Departure:` trailer.
No `warnings` came back on any digest.

## Verify gate findings

- Attempt 1 of 3 failed on `github.com/Knatte18/loomyard/cmd/lyx.TestCLIForm_EveryCommand`.
  The findings file noted that no recorded card touched a failing package.
  One fixer fork was spawned.
- Attempt 2 of 3 failed on `internal/reedcli` (`TestStrandNamingAndList/HubWideDirectoryAndDrift`, `TestNaming/TaskWorktreeStrandNameTitleAndEnv`, `TestNaming/EmptySlugGivesShortnameAndRole`, `TestNaming/RepairNamesRestoresAHandChangedTitle`, `TestOwnWindow_RealTmux/ShellOnlyStrandReadsDeadPastTheGraceAndResumeRelaunchesIt`) and the `internal/reedengine` package.
  The findings file noted that no recorded card touched a failing package.
  `ly:orch` named it a known main defect and briefed the fixer fork with main's tab-separator change.
  One fixer fork was spawned.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `81788dfa09965147b27f6855bff6adabcba71c4c` fix: trim lyx loom decision add help under the 2000-byte Long cap
- `b9b92feb5b473426908a4f7a12071bbe1894d19e` fix: split the reed pane tail on a tab, which tmux 3.4 passes through verbatim
