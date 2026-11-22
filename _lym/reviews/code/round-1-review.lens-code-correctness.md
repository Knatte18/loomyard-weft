```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function, branch and error path the diff adds or changes on a concrete input.

**List**
- `CheckCampaign`, no packages: the copies, the directory check and the tier-test check are all skipped, and it passes. Correct as specified.
- `CheckCampaign`, packages named with template tier-0 keys: copies run, `len(Copies.Tiers)==1`, and `HoldsTierTest` runs against `[0]`. Correct.
- `CheckCampaign` when the hub has no `gate.yaml` (`hardenPreflight` tolerates `ErrConfigAbsent`): `gateCfg` is zero, so `hardenLiveKey` returns `LiveKey{}` for both keys and every key check passes. Preflight passes, but the live gate's `lym gate test --gate harden-live` is later refused for the missing config. **Finding 2.**
- `copiesRun` with `(false, 3, nil)` returns false; with `(false, 3, [pkg])` it returns true; with `(true, 3, [pkg])` it returns false. Correct.
- `hardenLiveKey`, key absent: returns `LiveKey{}`. Key `{tiers:[2]}` with tier2 covering `[1]`: returns `Tested [1,2]`. Correct.
- `Campaign.SeedBrief`, packages set: lists them and never calls `Entry`. Correct.
- `Campaign.SeedBrief`, no packages with nil labels: "labels are none". Correct.
- `Campaign.SeedBrief`, `Entry` error: wrapped and returned. Correct inside the function, but see the next item.
- `RenderSeedPrompt`, `SeedBrief` error: returns a `promptRenderError`. The caller `runSeedSpawn` logs it with `logger.Warn` and returns nil (`bouncer.go:1811-1814`). `seedCall` then calls `ensureFocus(1)`, which writes the empty-lists fallback focus file (`bouncer.go:1762-1767`). Round 1 runs with no charter, and the not-found way forward naming `hardener.packages` (`hardenwiring.go:179`) never reaches anyone. **Finding 1.**
- `RenderSeedPrompt`, `SeedStencil` empty with `SeedBrief` set (a recipe with `harden_round` but no `seed_stencil`): the default template is read, and the extra `seed_brief` value is ignored because `FillWith` checks only unfilled markers (`stencil.go:55-63`). Correct.
- `RenderSeedPrompt`, `SeedStencil` named with `SeedBrief` nil: the fill fails on the unfilled `seed_brief`. Correct, and degraded the same way as above.
- `NewBouncer` with an unknown `SeedStencil`: construction fails, naming it. With an empty one, nothing is probed. Correct.
- `bouncerEntry`: an absent `seed_stencil` gives "" and the default template; the key is in the reject list. `harden_round` sets `SeedBrief` from `env.Harden`, which `requireHarden` guarantees is non-nil. Correct.
- `hardenCampaign.Entry`: an `OpenHub` error is wrapped; a `GetTask` error is wrapped; not-found gives the refusal with a way forward. Correct in itself; how that refusal is consumed is Finding 1.
- `NewLiveGate`, no packages, passed with `Started 0`: green, copies skipped. Passed false: red, with the failing lines deferred. `Refused` set: returns an error. Packages named with `Started 0`: red, as before. Correct.
- `sandboxDeps(singleRepo=true)`: `Seed` returns `(origin, "")` and `Clone` ignores the records bare. `primeCwd` is set through the shared variable, so `CommitOverride` reads the single-repo prime. On a clone failure, `CloneAndWire` returns `CloneResult{}` (`HubPath` ""), `NewSandbox` removes the container along with the seeded origin, and the error is returned. Correct.
- `newVerb`: `single_repo` in the JSON reflects the flag; `NewSandbox` errors go to `output.Err` with exit 1. Correct.
- `SeedLocalOrigin`: a failure in `initQuietBare` or `commitReadmeOnMain` is returned, and `NewSandbox` removes the partial bare with the container. Correct.
- `SeedLocalBares` refactor: the warp bare goes through `seedReadmeBare` and the weft through `initQuietBare`. Behaviour is unchanged.
- `CloneAndWireFromLocalOrigin`: passes `SingleRepo` with `WeftURL` and no `WarpURL`, which `CloneHub` requires (`clone.go:177`). Correct.

**Findings**

1. MEDIUM | design | `internal/shedadapters/bouncer.go:1925-1928`, with `internal/loomcli/hardenwiring.go:179`
   - **Summary:** a `SeedBrief` failure, including the "board holds no entry … way forward: name the modules … hardener.packages" refusal, is swallowed. `runSeedSpawn` only logs a warning, and `ensureFocus(1)` then writes the empty-lists fallback, so round 1 runs with no charter. The landing summary's claim that such an entry "fails the seed brief with a way forward" is not borne out.
   - **Evidence:** `bouncer.go:1811-1814` (render error logged, nil returned); `bouncer.go:1762-1767` (`ensureFocus(1)`, then Stuck without a reason).
   - **Fix:** probe `SeedBrief` before the seed call, for example in `CheckCampaign` or at `NewBouncer`, or make a seed-brief error Stuck with its reason instead of degrading.

2. MEDIUM | design | `internal/loomcli/hardenwiring.go:296-307`, `internal/hardenshed/campaign.go:56-111`
   - **Summary:** with no hub `gate.yaml`, `hardenPreflight` ignores `ErrConfigAbsent` and passes an empty `LiveKey` for both keys, and `CheckCampaign` now passes. Before the diff, the `HubSetsGate` check refused at Preflight. The run now spends a round's review and fix tokens before the live gate's `lym gate test --gate harden-live` is refused for the absent config. That contradicts `campaign.go:1`'s "refusals a run meets before any agent spends tokens".
   - **Evidence:** `gateslot/config.go:227-232` (absent file returns `Config{}` and `ErrConfigAbsent`); `hardenwiring.go:298-300`; `live.go:176-178` (refused call returned as an error).
   - **Fix:** refuse `ErrConfigAbsent` in `hardenPreflight`, or refuse a live key with no tiers in `CheckCampaign`.

Tool calls:
1. `stencil.Fill` and `boardengine.OpenHub`/`GetTask` signatures: to check extra-key handling and whether the board needs closing, which the manifest did not show.
2. `stencil.go` `FillWith` body and `bouncer.go` `runSeedSpawn`: to see how a seed render error is consumed.
3. `bouncer.go` `seedCall`: to see what happens after a swallowed seed render error.
