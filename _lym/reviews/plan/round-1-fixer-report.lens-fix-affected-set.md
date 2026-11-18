```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set searches behind the review's set-premise findings and check that the fix covers every member.

Unchecked: F6 (doc sentences each card falsifies, a set I can't search exhaustively), F10, F12, F13, F2, F4, F9 (no set premise, or only one site).

Checked and covered: F1, F5, F11.
- **F1:** every `battencli.ShuttleRunner` call (`beamcli/dobbydeps.go:835,853,862`) and the cancel caller (`cancelcmd.go:275`) go through `primeShuttleRunner`, which card 3 now edits.
- **F5:** the only `adoptDeps{` literals are `adopt_test.go:125` (`world.deps`, now in card 5) and `deps.go:57` (`hubDepsAs`, already in card 5).
- **F11:** `strandRemovable` has three callers: `stop.go:148` (the stop verb), `stepstop.go:99` and `attach.go:236`. Card 12 names all three.

Finding:

- **BLOCKING** — card 2, the added Intent sentence "`configreg`'s `TestModules_EveryTemplateKeyClassified` pins every module's machine keys, so its `wantMachineKeys` gains a claude case wanting `[]string{"account_pins", "accounts"}`."
  The same test also pins claude's `LocalDenied`, and card 2's machine keys land there too. The fix covers only `wantMachineKeys`, so the test still fails once card 2 lands.
  Evidence:
  - `internal/configreg/configreg_test.go:161` has `"claude": {"binary", "deny_ask_user_question"}` in `wantLocalDenied`.
  - The test asserts it at `configreg_test.go:192`, with `slices.Equal(m.LocalDenied, want)`.
  - `internal/configengine/layers.go:172-174` defines `LocalDenied` as every path classified outside `CategoryNone`, so it includes the `CategoryMachine` keys `accounts` and `account_pins`. Codex's `sandbox_mode` already sits in both lists (`configreg_test.go:162` and the `wantMachineKeys` codex case).
  - Fix: the sentence also needs `wantLocalDenied["claude"]` to become `{"account_pins", "accounts", "binary", "deny_ask_user_question"}`.

Tool calls:
1. Grepped the callers of `ShuttleRunner` and `primeShuttleRunner`, tests naming `MachineKeys` or `account_pins`, `adoptDeps{` literals and `strandRemovable` callers. These are the full sets; my context held only the review's named members.
2. Grepped `configengine/layers_test.go` and the configreg tests for claude key pins, to find other tests pinning claude's keys, which my context lacked.
3. Read `configreg_test.go:150-185` and the `LocalDenied`/`MachineKeys` definitions in `layers.go`. My context showed only the `wantMachineKeys` part of the test, not the `wantLocalDenied` map or the category rule.
