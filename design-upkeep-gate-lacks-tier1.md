# Upkeep gate call is refused because gate.gates.upkeep lists no tier covering tier1

`upkeep-gate-lacks-tier1` · Task · bug, gate, config · starting · Open-Window

## Seen (board tidy, 2026-11-27, main at `2326ef4da`)

`lym gate test --gate upkeep` was refused, so it ran no check:

```
gate test: --gate "upkeep" lists no tier that is or covers tier1, so its marked tests there cannot run: TestCSharpFixtures_CompileWithDotnet, TestGateRedFirst_Dotnet, TestGateTest_DotnetFixture, TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain; set gate.gates.upkeep to list tier1; way forward: fix it with "lym config gate" from the prime; a session that is not the prime takes none of these steps; it reports status: FAILED and the orch runs them
```

## Fix

Set `gate.gates.upkeep` to `{tiers: [0, 1], scope: marked}` with `lym config gate` from the prime, as the change description of `2326ef4da` asks under "To check by hand".
Every binary that reads the hub's gate config must hold that change first, since an older one does not accept the scope `marked`.
Then run `lym gate test --gate upkeep` on the main branch and confirm that it runs the dead-code and .NET tests.

The shipped template gives `{tiers: [0], scope: marked}` while the marked upkeep tests are `tier1`,
so a hub whose `tier0` does not cover `tier1` is refused the same way until it sets the key.

## Task (operator, 2026-11-27)

Change the shipped gate template so `gates.upkeep` defaults to the tiers that hold the marked upkeep tests, `[0, 1]` today, or derive the tiers from the marked tests so no hub needs the key by hand.
Setting this hub's key and the hand check on main stay with the orch after the operator's next lym update.
