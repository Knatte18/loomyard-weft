```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out (F1 a–f, F2 and F3) by looking for a concrete case where it goes wrong.

1. **BLOCKING.** Changed text: card 13, the new paragraph "`InnerRunDeps.Sleep` is deleted with `NewInnerRun`'s `waitOrCancel` default, and the producer's `pollInterval` field with it".
   Summary: card 13 deletes a field that code only card 14 changes still reads, so the package stops compiling at card 13.
   Evidence: `internal/battenshed/innerrun_pause.go:58` reads `p.deps.Sleep(ctx, p.pollInterval)` inside `holdSpawnWhileCovered`.
   Card 13's Edit list does not hold `internal/battenshed#innerRunProducer.holdSpawnWhileCovered`; card 14's does, and card 14 comes after card 13.
   So once card 13 deletes the `Sleep` field and the `pollInterval` field, battenshed does not build, and card 13's own tier0 gate fails.
   The review's premise said as much: Sleep "is read in two places today, the `wait` loop that card 13 deletes and the spawn hold that card 14 turns into a yield".
   The proposal still put the deletion in card 13.
   Fix: move the deletion of `InnerRunDeps.Sleep`, the `waitOrCancel` default and the `pollInterval` field into card 14, together with the matching test fills, the `nilLegal` entry and the `innerRunEntry` doc.
   Another way: card 13 also lists `holdSpawnWhileCovered` and drops its sleep there, but that overlaps card 14's change.

2. **LOW.** Changed text: card 7's Tests addition, where the `BoardClaimIsClearedWhenBattenNeverSeededTheRun` row now asserts `draft`.
   Summary: the row's name and its error text "want it cleared" still say the claim is cleared, but it is now set to `draft`.
   Evidence: `internal/pairteardown/teardown_integration_test.go:384` and `:391`, where the message reads `t.Errorf("board status after Run = %q, want it cleared", ...)`.
   A reader would notice; the card could rename the row, for example `BoardClaimIsLeftDraftWhenBattenNeverSeededTheRun`.

I found no case that breaks F1 (c)–(f) or F3.
yaml.v3 belongs on the allowlist the way battenshed's list holds it.
Moving `proc.Detach` and the directive imports agrees with what the code does today.
`PassExit` is created in card 33 before card 37 uses it.

Tool calls:
- Grepped `00-overview.md` and cards 13–16 for "in-pass", to see which in-pass waits remain once the seam is gone. The overview was not in my context.
- Grepped `00-overview.md` for D4. It holds nothing, so I had to find where the decision lives.
- Listed `_lyx/` and grepped it for the D4 heading, to find the decision record.
- Read D4's lines on waits and sleeping in `_lyx/discussion/decision-record.md`, which was not in my context.
- Grepped battenshed's production code for every `deps.Sleep` and `.pollInterval` reader. The parent's earlier grep covered only `innerrun*.go` and did not show who owns `innerrun_pause.go:58`.
