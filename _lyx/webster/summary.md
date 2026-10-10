# Gate test scope: slotted gates, tiered verification, Publish failure record

All four execution batches recorded `status: done`; each batch holds six cards, one commit per card.

## Batch 01 (cards 1–6), head `8422ab71e`

Built the hub-wide slot pool `internal/gateslot` (file-lock slots, holder records, inheritance, GOFLAGS cap, wait records) and its hub-wide `gate` config module.
`verifyrun.Run` now takes the shell's environment, and `verifytree.Verify` acquires a slot, marks the wait and starts its timeout once held.
`hubgeom` gained the pool constructor and webster geometry's `GateSlots`; webster's gate, `lyx webster verify` and the card-verify rerun take the slot.
Deviations: `internal/testkit/envkit/envkit.go` (`GateSlots` nil-legal) and `cmd/lyx/testdata/test-budget.yaml` (`internal/websterengine` +1).

## Batch 07 (cards 7–12), head `b2e5617cc`

Publish and Finalize verifies, the Webster-Burler round gate and the plan gate's caller-uncovered go list take the slot.
Added the `lyx gate test` module (`internal/gatecli`), `lyx loom status` gate-wait and holder display, and the standing agent deny against raw module-wide and tmux/llm-tier go runs.
Deviations: `envkit.go`, the `seam_enforcement_test.go` import allowlists of `landingshed`, `loomshed` and `shedrecipe`, `cmd/lyx/seamsignature_test.go`, and new `internal/gatecli/cli.go` and `run.go`.

## Batch 13 (cards 13–18), head `f53f6b572`

The plan gate refuses a module-wide card Verify; the card gate runs `lyx gate test` over the card's own packages, and a once-per-batch integration gate exists.
The verify-fix stencil and the CLAUDE.md push rule name `lyx gate test`; `lyx config` refuses hub-wide writes from pairs and pair-spawned sessions; landing is hub-wide.
Deviations: `internal/testkit/plankit/plankit.go` and `cmd/lyx/testdata/test-budget.yaml`.
The fork reported appending to a few files with shell heredocs, against the no-scripted-edits rule.

## Batch 19 (cards 19–24), head `aa76dfb0d`

Added the Publish failure record codec in `verifytree`; Publish writes and clears it; the round gate runs the failing tests and a tmux pass while a record is present; the Webster-Review rubric renders the checked record.
The comment lint passes comma-plus-conjunction breaks only before a clause, and PATTERN records the gate table, the llm decision and the slot entry.
Deviations: `envkit.go`, `internal/loomcli/wiring_seams_test.go`, `internal/landingshed/finalize_verify_test.go`, `internal/loomshed/seam_enforcement_test.go`, `internal/impactset/derive_integration_test.go`.
The fork reported one `cat >>` heredoc append, against the no-scripted-edits rule.

## Open notes

`gofmt -l` lists files the forks did not touch, such as `internal/fabricengine/pull.go`.
The plan-level verify is left to the gate.
