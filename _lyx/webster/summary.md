# Self-reported issues say something

All six batches recorded `done`, in order, with no recovery needed.

1. `01-remove-tier1-filing` (`fcb9738d7`): loom's GitHub filer, its filed marker, its seam and the issue rendering are gone, together with their tests.
   One reported deviation: `internal/loomcli/gotoentry_test.go` was changed to match the new `DetectCrashResume` return, though it was outside the card's file-ops.
2. `02-retire-selfreport-key` (`f8fe09cdd`): the `selfreport` config key is retired, and the entry observation follows Tier 2's on/off.
   A retired-key test loads the template plus `selfreport: false` and asserts the config equals the clean template.
3. `03-rename-handoff-voucher` (`a7f691200`): the step clean-handoff marker became the handoff voucher (`handoff-voucher.json`), and the self-report files were renamed with `git mv` (`entryobservation.go`, `entryobservation_test.go`, `handoffvoucher_test.go`).
4. `04-anomaly-friction-notes` (`cf8ff2c88`): the `loom-halt` note carries the anomaly kind and recent history, and run and step entry write a `loom-crash-resume` note.
   `CleanStepHandoff` became `Vouched`.
5. `05-start-vouches` (`091324f4c`): `lyx loom start` writes the handoff voucher right before it spawns a driver, only with Tier 2 on, and warns without failing when the write fails.
6. `06-agent-filed-issues-docs` (`809f36611`): `PATTERN-agent-filed-issues` was added with its scan test in `cmd/lyx/agentfiledissues_test.go`, and the reflection stencil, friction docs and stencil claims were updated.

No deviations were reported beyond the one in batch 1.
The plan-level verify gate runs when this turn ends.
