```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each set-premise finding (the fix-affected-set lens).

Unchecked: none. F1, F2 and F9 have set premises, and I reran each one's search. The other findings name a single site or file, so they have no set to rerun.

no findings

- **F1** (callers of `runAgent` in driftengine): `verify.go:476` (`judge`) and `verify.go:1072` (`passCommand`) are both covered by card 6 now. The third call, at `expect.go:147`, is the expect agent's own `runAgent`, which runs on a different path with no parent agreement entry. It is outside D3.
- **F2** (shedrecipe fixtures that build pawl deps): `validPawlDeps` (`fixture_test.go:67`) is the only builder of `pawlengine.Deps` outside pawlengine. `newTestEnv` uses it (`fixture_test.go:315`), so card 15's two shedrecipe targets cover the set.
- **F9** (goto and relaunch sites): loom's sites are `drivewiring.go:282` (tackler relaunch, in `goDriverDepsFor`), `drivewiring.go:346` (`driverGoto` through `RunGoto` over `c.spec`, which `specFor` sets), `arbiterconsume.go:22` and `shedverbs/goto.go:34`. Card 14 covers all four. The battencli sites (`drive.go:219`, `:320`, `:602`) are batten's own and leave `Admitted` nil, which the card says calls nothing.

Tool calls:
1. One grep for the call sites of `runAgent`, the builders of pawl deps and the constructors of `GotoRequest`/`RelaunchRequest`/`RunGoto`. My context held only the sites the review and the fixer had cited, not the full sets.
