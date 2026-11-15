```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it or record a gap.

**Per-behaviour list**

1. Go still picks the gated fragment when `GateFixed` is set, and the ungated one otherwise. `makeChangeList` is unchanged, but the gated marker line changed. It is pinned by `TestDarnSpec`, rows "first spawn renders the prior-work token as none" and "a cold fixer on the verify gate has the writer run only a reported failure's test", through `gatedNoGateStep` and `ungatedTestStep`. It is also pinned by the `TestWire_DarnRecipe` subtest "the writer's prompt drops its gate runs only for a pair run whose gate_fixer holds a model-spec". Both constants had to change: the old line ended `yourself.`, so the old value no longer matches the new `yourself, bar one run.`. This is a needed sync, not redundant coverage. Covered.
2. The gated fragment still carries its one marker, `subagent_model`. Pinned by `contracts/stencils/markers_test.go:64`. Covered.
3. The new item-3 wording: one run of the reported failure's test before diagnosis, `-- -run <TestName>`, and a pass routed to Step 3. This is static stencil prose, so it is exempt from coverage. The diff pins it anyway with `gatedReproduceRun`; see finding 1.
4. Doc comment changes in `spec.go`, `doc.go` and `stencils.go`. These are prose and need no test.
5. Subagents still run no gate. The diff leaves this prose unchanged, and it is exempt.

No behaviour the diff adds lacks a covering row.

**Findings**

1. **BLOCKING · `consistency`** at `internal/darnengine/spec_test.go:77`, plus its uses at `:104` and `:112`. The new constant `gatedReproduceRun` asserts a stencil's static prose. It holds "When the task starts from a reported test failure, run that test once before you diagnose it". The test asserts it in the gated row's `wantInPrompt` and in the ungated row's `wantNotInPrompt`.
   - Evidence: `pattern/PATTERN-test-speed.md:248-252` says "No test pins a stencil's static prose". A stencil test may pin only a marker set, a Fill round-trip, a value Go fills, reads or names, a fragment Go chooses, or a rule across a stencil family.
   - Why it is not exempt: which fragment Go chose is already told apart by `gatedNoGateStep` and `ungatedTestStep`. The extra line pins wording and adds no signal about which fragment was chosen.
   - Rubric: Test economy, "a new assertion on the static prose of a stencil, spec, doc or other text".
   - Fix: drop `gatedReproduceRun` and its two uses, and leave review to guard the wording.
   - Severity: BLOCKING because it breaks a background rule of a listed constraint, `PATTERN-test-speed`. If the handler reads that bullet as advisory test economy, it would be MEDIUM.

**Tool calls**

- grep of `contracts/stencils/markers_test.go` for the gated fragment: the manifest listed that file as a reader of the fragment without showing what it asserts.
