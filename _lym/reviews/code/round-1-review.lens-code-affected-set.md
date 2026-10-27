```yaml
lens: code-affected-set
outcome: reported
```

Task: list what the `AwaitRow`/`WaitCapture`/`capture: wait` removals and the `ReadJSON` merge force outside the touched lines, and judge each member covered or not.

**Per-change set**

1. `Deps.AwaitRow`, `VerbStatus.AwaitRow`, `Row.WaitCapture` and `CountedCapture`'s `awaitRow` parameter.
   - The set is the definitions, the `driftDeps` and `verbStatus` wiring, the three `CountedCapture` callers and the tests.
   - A repo-wide grep (excluding `_lym`) for `AwaitRow|WaitCapture|awaitRow` returns nothing.
   - The remaining `CountedCapture` callers use the three-argument form: `verbs.go:173`, `verify.go:164` and `records_test.go:36`.
   - `go build` and `go vet` of `driftcli`, `loomcli`, `driftengine` and `shedrecipe` are clean.
   - Covered.
2. The `capture: wait` config key and the `wait` mode.
   - A grep for `capture: wait`, `capture mode` and `wait mode` finds no recipe, doc or test still using the mode.
   - The `"wait"` hits are all unrelated: `activity.wait` fields, `--wait` flags and a `waitDir` test variable.
   - Covered.
3. The `verbStatus` seed read.
   - The seed read, its `ReadSeed` use and the not-found error are gone from `internal/driftcli/wiring.go`.
   - The doc comment above it matches the new body.
   - The `shedrun` import is still used for `StatusFile`, `StatusLock` and `SelfRunID`; the build passes.
   - Covered.
4. The `ReadJSON` merge.
   - `ReadJSONNoMkdir` has no remaining reference anywhere.
   - The comment on `segmentBounces` (`internal/loomcli/wiring.go`, the line after `state.ReadJSON`) still says "A run directory not yet created fails the lock open with not-exist".
   - That comment is the parent's merged text, not a line the task changed, and the rubric excludes the settled remainder and the parent's change from this review.
   - I do not file it.
5. Docs and the recipe.
   - `contracts/recipes/darn-recipe.yaml`, `internal/driftengine/doc.go:71` and `docs/overview.md:498` and `:564` describe Drift-Baseline detaching and Drift-Await holding the run, and agree with each other.
   - The `internal/driftcli/cli.go:260` and `:459` help text and `internal/driftcli/doc.go:19` name a Drift-Await halt without a loom-only qualifier.
   - Covered.
6. Stencils.
   - The grep for wording about darn's capture wait or a loom-only Drift-Await finds none.
   - Covered.

**Findings:** none.

**Tool calls (1):**
- `git grep` for stale wording about waiting captures and for Drift-Await mentions in `contracts`, `docs` and the `driftcli` and `shedrecipe` docs. My context lacked a docs and stencil sweep for stale prose.
