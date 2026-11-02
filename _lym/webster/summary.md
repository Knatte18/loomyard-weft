# One shared cold gate fixer for four gates

All five batches finished with `status: done`, with no recoveries, renames or pattern departures.
The scope guard listed files outside each batch's declared targets as deviations, all of them the new or edited files of the batch's own cards.

## Batches

- **01 gatefixer-core**: new `internal/gatefixer` with fixer roles, record and store, spawn-or-attach, wait, stop and the runproc registrar; the shared cold frame stencil and `RenderPrompt`; `fix-verify` and the plan fixer now wait on `gatefixer`, and the old webster cold prefix and wait helpers are gone.
  Deviations: gatefixer sources and tests, `webstercli` wiring and `fixverify.go`, `websterengine` `render.go`, `template_test.go` and `verifyfixer.go`.
- **04 webster-file-headers**: file headers renamed to the shared frame; gate.yaml `fixer` on the `card` key and `fixer_timeout_min`; the `darn.gate_fixer`, `tapestry.plan_gate_fixer` and `loom.discussion_gate_fixer` keys; `gatefixer.WrapRowGate` with the row fixer's record set.
  Deviations: `darnengine` and `loomengine` config tests, `gateslot` config and its test, gatefixer row-gate files.
- **08 shedrecipe-fixer-element**: the `fixer` gate element key, the `Env.GateFixer` seam and the three rows' opt-in; `loomcli` wires the row fixer seam, visit directory and body stencils.
  Deviations: `loomcli` gatefixer files and seam test, `loomshed` gates, `shedrecipe` entries.
- **10 goto-ends-fixers**: an admitted goto or targeted relaunch ends every fixer role under the pair first; Plan-Write and Discussion-Write prompts drop the validator loop when the fixer is on.
  Deviations: `contracts/stencils/markers_test.go`, `loomcli` fixer files, `loomengine` discussion, plan and twill files, `twilldirective` and its test.
- **12 card-gate-front**: the front carries the strand's role and keys a fixer's own call apart; the detached runner's fixer phase, attach, re-run and join rule; the Webster fork and recovery prompts name the card-gate fixer when it is on.
  Deviations: `gatecli` card-fixer and detach files and tests, `websterengine` `render.go` and `template_test.go`.

Every batch passed its card gates and the batch gate at tiers 0 and 1.

## Verify gate findings

The verify gate failed once on `internal/configreg.TestModules_EveryTemplateKeyClassified/darn`, and no recorded card touched a failing package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `a84d2f2f0f8d6672cd7f17d8ef1d5240e087d6ef` fix: list darn's gate_fixer among the hub-wide local-open keys the classification test expects
