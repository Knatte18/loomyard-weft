# Fixer report — plan, round 1

Every finding's premise held against the code; every finding is fixed.
Card numbers below are the plan's numbers after this round: the old card 12 (`verifytree-unslotted`) is now card 14, and the old cards 13 and 14 are now 12 and 13.
`lyx loom validate-plan` passes with informational findings only.

## Changes

- **F1** (fixed as proposed): card 10 targets `loomCLI.wire` and `loomCLI.wireDarn` and fills the `round` seam there from `gateslot.LoadConfig` over the pair's config dirs and `TierList("round")`; card 13 (landing) no longer claims to build the seam.
- **F2** (fixed as proposed, extended by the self-check): card 2 targets `configcli#dirsOf`, `configDirs.selectedLayer` and `TestDispatchLayers`.
  `dirsOf` takes its board from `fabricengine.ConfigDirs(l).Board`, so the board is empty outside a hub, and `selectedLayer` maps an empty `--layer` on a hub-wide module with no board to `LayerWorktree`.
  `TestDispatchLayers` gains the standalone write-then-load row, and the ImpactSummary names the behaviour change.
- **F3** (fixed as proposed, first option): moved `verifytree-unslotted` after the landing card with `lyx loom remove-card 12` and `lyx loom insert-card 14`, then rewrote its body as card 14.
  It now states that every site already renders its gate call, and it owns the `PATTERN-verified-tree.md` slot-line removal, which card 13 no longer claims.
  Cards 12 and 13 say they still pass today's pool and timeout to `verifytree.Verify`.
  Every renumbered prose mention in cards 4, 10, 12 and 15 is updated.
- **F4** (fixed as proposed): card 9 runs the guard step for a computed, `--base` or card scope, names the guard packages per form, and has the `--base` row assert the guard step through `guard_tier`'s slots.
- **F5** (fixed as proposed): card 9's two-module row is now a `--card` run over a card spanning the root and a nested module, with the timeout killed across both invocations; a new computed-scope row covers a nested-module file falling back to whole, naming the reason.
- **F6** (fixed as proposed, set widened): card 19 targets `singleSlotLimits`, `fixedLimits`, `TestAcquire_ReadsLimitsAtEveryAcquire` and, from the self-check, `templateDefaults`, each dropping `GoParallel` in its Forced edits.
  Card 14 states that `TestVerify_DropsStrandName` drops its slotted row, which is that test's `GoParallel` literal.
  `fixedLimits` left card 3's Uses list, because `validate-plan` raised `uses-later-target` once card 19 targets it.
  Card 19 only drops a field from the literal, so card 3's tests call the helper unchanged.
- **F7** (fixed as proposed, set widened): card 13 targets `configcli#TestDispatchHubWideBoard`, whose `darnTreeVerify` becomes `lyx gate test --wait --tier 1 {targets}`.
  From the self-check, darn `TestLoadConfig`'s passing `lyx gate test {targets}` rows and `TestDarnTreeScope_ClassifiesAtEachCall`'s `scopedVerify` move to the same form.
- **F8** (fixed as proposed): card 4's `TierTimeoutLine` and card 14's landingshed Stuck reason name the whole-tier `--set 'tiers.tier<N>={...}'` step; `TestWatchdogLine` and `TestVerifyGate_CheckMapsVerifyResult` assert it.
- **F9** (fixed as proposed): card 11 gives `lyx gate card-verify` `clihelp.AudienceRole`, and `TestGateCardVerify` asserts the annotation.
- **F10** (fixed differently): card 5 re-signs `DeriveTargets` and `changedPathsInTree` with `ctx`, and targets `listPackages`, re-signed to take `ctx` and start `go list` with `exec.CommandContext`, plus `Derive`, which passes `context.Background()`.
  Card 9's `computeScope` passes the pre-tier ceiling's context.
  Not made: the proposed claim that "its git reads stop when it is cancelled".
  `gitexec.runCore` (`internal/gitexec/gitexec.go:190`) starts local subcommands with a plain `exec.Command` and exposes no context, so that claim would be false.
  The card says instead that `DeriveTargets` checks `ctx` before each git read and that an in-flight local read runs to its end.
- **F11** (fixed as proposed): card 14 targets `shedrecipe#Env`, `loomCLI.wire` and `TestWire_DefaultConfig`, and deletes `Env.GateSlots`, its wire line, its assertion and envkit's `nilLegal` `GateSlots` entry.
- **F12** (fixed as proposed): card 19 targets `docs/benchmarks/running-tests.md`, moving its tier section to `tier0`–`tier3` and its gate lines to `--tier <n>`.
- **F13** (fixed as proposed): card 3's bound sentence names each holder's bound and says a drain waiter queued behind a `cmd/testtiming` run waits for it.
- **F14** (fixed as proposed): removed card 8's `--tier 0 --tier 1` covered row from `TestGateTest_Scenario` and card 9's "skipped when the call lists `tier1`" row.
- **F15** (fixed as proposed): card 15 cuts `TestCardGateCommand` to the gate-call-plus-lints row and the dangling-list refusal.
- **F16** (fixed differently): card 16 renames `VerifyTiersValue` to `BulletListValue` itself, as a Create of `plan:internal/languageconfig#BulletListValue` and `TestBulletListValue` with a Delete of `VerifyTiersValue` and `TestVerifyTiersValue`.
  Card 16 already rewrites every caller, and `doc.go`'s two mentions are named.
  Not made: the proposed `Rename` card after card 16.
  The hub's `language.yaml` (`_board/_lyx/config/language.yaml`) sets no `rename_tools`, and the template default is `[]`.
  So the plan gate raises the blocking `rename-tool-missing` for any symbol `Rename` pair (`contracts/specs/loom-plan-spec.md`, the `rename-tool-missing` paragraph).
  I inserted and then removed the card with `lyx loom insert-card 17` and `lyx loom remove-card 17`, and restored the renumbered mentions.

## Self-check

- fix-claims, MEDIUM: card 2's `selectedLayer` branch can never fire, because `dirsOf` always sets a board (`hubwide.go:81`, `fabricengine.BoardDir`). Fixed as proposed: card 2 targets `dirsOf`, which takes its board from `fabricengine.ConfigDirs`.
- fix-proposal, MEDIUM: the same finding as fix-claims'. It is merged into that entry and fixed by the same change, which also says the hub-file write, guard and commit run only for `LayerDefault`.
- fix-affected-set, MEDIUM: `templateDefaults` in `gateslot/config_test.go:16` keeps `GoParallel`. Fixed as proposed in card 19.
- fix-affected-set, MEDIUM: darn `TestLoadConfig`'s `lyx gate test {targets}` rows and `TestDarnTreeScope_ClassifiesAtEachCall`'s `scopedVerify` lack `--wait`. Fixed as proposed in card 13.
- fix-affected-set, LOW: `unusableGatePool` may set `GoParallel`. Disputed, premise false: its `Limits` closure returns `cfg.Limits()` with no literal (`internal/planglyph/glyphchain_test.go:1045-1060`), and its `go_parallel: 1` seed loads once the key is retired.
- fix-consistency, LOW: card 10's sentence still credited the computed scope with the round gate's guard coverage. Fixed as proposed: it names the round call's `--base` scope and its guard step (card 9).
- fix-consistency, LOW: card 16 did not name `languageconfig/doc.go`'s `VerifyTiersValue` mentions. Fixed as proposed.

## Disputed

None of the review's findings.
One self-check finding is disputed, the fix-affected-set LOW above.

## Deferred

Nothing deferred.
