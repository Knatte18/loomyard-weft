# Card 25 — shedadapters-shedfake

**Edit:**
- `shedadapters/attachprobe_test.go`
- `shedadapters/bouncer_clear_test.go`
- `shedadapters/bouncer_commit_test.go`
- `shedadapters/bouncer_config_test.go`
- `shedadapters/bouncer_judge_test.go`
- `shedadapters/bouncer_replay_test.go`
- `shedadapters/bouncer_seed_test.go`
- `shedadapters/bouncer_skip_test.go`
- `shedadapters/bouncerprompt_test.go`
- `shedadapters/burler_test.go`
- `shedadapters/singlellm_test.go`
- `shedadapters/webster_test.go`

**Uses:**
- `testkit/shedfake/shedfake.go`

**Intent:**
Move shedadapters' tests onto `shedfake` (#324 §2): its `Shuttle` fake is the copy the kit was built from, so this package converts first.

Delete `fakeShuttle`, `evalGateList` and `fakeShuttleWithAttachHook` from `singlellm_test.go` and `fakeBurlerRunner` from `burler_test.go`; tests use `shedfake.Shuttle` and `shedfake.BurlerRunner` with the exported field names.
A wrapper adding behaviour on top of the shuttle (an attach hook, a per-spec answer) becomes a `RunFn`/`DuringAttach` setting, not a new type.

Every `p.Call(context.Background())` followed by the fatal-on-error and outcome-compare block becomes `shedfake.RequireOutcome(t, p, want)` (or `shedfake.CallOK` where the outcome is checked differently); a site asserting an error from `Call` keeps calling `Call` directly.
The returned `OutputPointer` keeps feeding whatever the test asserted on it.

The bouncer fixture consolidation is card 26 and the log-capture helper card 36; leave both as they are here.
Apply D10 to every touched test.

**ImpactSummary:** Untagged shedadapters tests only; two local fakes and the Call boilerplate move to shedfake, every scripted value and assertion kept.
