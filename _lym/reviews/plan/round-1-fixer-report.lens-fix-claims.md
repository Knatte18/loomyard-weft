```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checked that each sentence the fix added holds against the code at HEAD.

**MEDIUM** — card 11, the new Choice (D2) sentence "Bound: an in-process call now waits for its slots without a cap, ended only when a holder releases its slots, each holder's run being bounded by its tier's or build step's `timeout_min` …"
- **Summary:** the bound names only gate tiers and the build step, but other kinds of holder also take slots from the same pool. Their runs are bounded some other way, or the code shows no bound.
- **Evidence:** other code that takes a slot:
  - `internal/cardverify/runner.go:167`: a line run inside a held slot, bounded by `Runner.Timeout` (`DefaultTimeout` is 10 minutes, `runner.go:20-21`), not by `timeout_min`.
  - `internal/driftengine/steprun.go:149`: drift step runs.
  - `internal/planglyph/typesload.go:126`: plan-glyph type loads, which acquire with `context.Background()`.
  - `cmd/testtiming/resources.go:153`.
- **Fix:** the sentence should cover those holders or name the bound per kind of holder, for example "each holder's run being bounded by its own timeout, a tier's or the build step's `timeout_min` and a slot-held verify line's runner timeout".

These claims hold:
- **Card 10:** `state.UpdateJSON(path, lockPath, mutate)` matches `internal/state/state.go:148`.
- **Card 11, `current.json`:** writing it through `state.WriteJSON` with its own lock path is consistent with `state.go:30-43`. `WriteJSON` takes `lockPath` internally, and the key directory's `lock` is a separate file, so nothing nests on one lock.
- **Card 11, the interrupt clause** ("exits 128 plus the signal number as today"): a cancelled wait goes `acquireSlot` → 1 (`run.go:558-559`) → `runWithSlots` halt (`run.go:463-466`) → `runTest`'s `exitAfterSignal` (`run.go:176`, `617-621`), which gives 128 + n.
- **Card 11, `cli_test.go`:** its only member is `TestGateTest_RefusesBeforeResolvingTheHub` (line 17). The header's slot-wait clause and the `shuttleengine` import are as the card states.
- **Card 13:** the ok envelope text matches `internal/gatecli/cardverify.go:96`.
- **Card 14:** a gate call without `--wait` fails `IsGateCall` (`runner.go:101-103`), so it runs as a slot-held line under `Runner.Timeout` (`runner.go:40-46`, `129-141`). That is the reason the card now gives.
- **Card 4:** the old header and `awaitVerb` comment text match `await.go:1-2` and `17-18`.

Tool calls:
1. grep of `gateslot/config.go` for `timeout_min` and of `cardverify` for its slot-held timeout: my context had neither the per-tier timeout field nor how cardverify bounds a line.
2. grep for `tierFile` and the tier timeout: confirming that tiers, not only the build step, carry `timeout_min`.
3. grep for every slot `Acquire` caller and read of `run.go:40-70`: finding every kind of holder the bound has to cover, and the path a cancelled wait exits through.
4. read of `run.go:455-475`, `driftengine/steprun.go:140-160` and `planglyph/typesload.go:115-130`: the halt path after a cancelled wait, and what bounds the other holders.
