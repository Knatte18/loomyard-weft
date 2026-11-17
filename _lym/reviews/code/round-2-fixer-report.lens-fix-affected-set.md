```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: reran the searches behind F1, F2 and F3 and checked that each fix covers every member of its set.

**Findings**

1. **MEDIUM** — `internal/gatecli/cardfixer_test.go`, new subtest `CodexKitRendersTheCodexPrompt` and the `engine: r.engine` kit stub.
   - **Summary:** The fifth F1 site, the derivation `engine: rig.engines.EngineName(model)` at `internal/gatecli/cardfixer.go:199`, is still unpinned.
   - **Evidence:** The `engine` the new subtest reads comes from the test's own `fixerRig.engine` field and is returned by the stubbed `cardFixer` seam. `newCardFixer` is never called in the tier 0 tests, so a literal or the wrong choice at line 199 still passes every test.
   - **What it covers:** The new subtest pins only the consumer, `detachfixer.go:270,277`, which passes the kit's engine through. F1's fix item (5) asked to assert the kit's engine against `rig.engines.EngineName(model)`.
   - **Gap:** `newCardFixer` needs a hub and `newStrandRig`, which `detachfront_integration_test.go` is the only gatecli test file to reach. The fixer's report should name this as a partial fix.

**Sites I checked**
- **F1, the other four sites:** each now drives a codex namer and asserts `PromptEngine` or the edit directive.
  - `shedarbiter`: both `settleFresh` and `attachAndConclude` are covered.
  - `sheddriver`: both `tackler.go:191` (attach) and `:247` (spawn) are covered.
  - `dobbyengine.spawnTidy`: covered.
  - `loomcli.startLLMDriverArm`: covered with an `engine` override on the fake starter, so the "choice's own engine" path is not exercised separately. That is acceptable.
- **F1, the callers the review called already pinned:** I did not re-verify these, and they are unchanged. They are `treadleengine`, `samplerengine`, `landingshed`, `frictionengine`, `mergeresolve`, `websterengine`, `weaverengine` and `loomcli/gatefixer.go:261`.
- **F1, not named by the review:** `shedadapters/bouncer.go:1687` and `:1926` (bouncer seed and judge). The review lists `shedadapters` as covered by the codex-models row, and I did not check that row. These two sites are the only candidates the review's list leaves open.
- **F2:** the set of adapters that gained engine resolution is `RunnerShuttle` (`sheddriver/agentshuttle.go:67`), `runnerTidyShuttle` (`beamcli/dobbydeps.go:692`), `runnerDriverStarter` (`loomcli/driverlaunch.go:79`) and the card fixer kit (`gatecli/cardfixer.go:199`). The new `summary.md` text names all four.
- **F3:** no `tacklerPrompt()` message remains in the Go tests. The only `tacklerPrompt*` hits are the `tacklerPromptFileName` constant and its `TacklerPromptPath` use in `internal/sheddriver/tacklerrecords.go`, which are unrelated.

**Tool calls**
1. A grep over `internal` for `EngineName(` callers and implementations, `tacklerPrompt` stale names, and test fakes. One call for all three sets: my context held only the five files the review named, not the full caller set.
