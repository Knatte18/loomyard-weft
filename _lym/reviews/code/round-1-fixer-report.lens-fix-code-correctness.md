```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed.

1. **MEDIUM** — `internal/gatecli/scopes_integration_test.go`, the last assertion of the `//lym:only` subtest: `wantRefusal(t, gate(t, pair, nil, "--gate", "upkeep"), "way forward")` over the broken `go.mod` passes with the defect in place.
   - It is meant to pin F8's "a failed `go list` refuses a marked key" branch.
   - Delete `|| isMarkedCall(cfg, request)` in `withOnlyTests` and the call warns and returns with `marked` nil.
   - `markedScope` then finds `go.mod` present and refuses with "no test of the module at … is marked //lym:only upkeep … way forward: …", still before any tier command, so the assertion holds.
   - It needs text only the listing refusal carries, such as the `list the packages of` of the `modulePackageDirs` error.

2. **LOW** — `internal/gatecli/only.go`, the new `markedScope` comment "so a pass always means every marked test ran": the code does not guarantee this.
   - `runTest` still applies `withoutHeld(sc.guards, request.held)` by bare name after `markedScope` has made both of its refusals.
   - Take `a.TestX` marked `upkeep` and `b.TestX` marked `finalize`. `--gate upkeep` puts `TestX` in `held`, drops `a.TestX` from the guards and exits 0, possibly having run nothing.
   - The review accepted bare-name matching as a bound, so the claim should be bounded, or the marked guards left out of `withoutHeld`.

I found nothing else in the changed branches or the changed `ModuleOnlyTests` and `modulePackageDirs` callers.

Extra tool calls: none.
