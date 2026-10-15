# Lens review

Four forks ran, one per lens; each returned a report.

- forced-edits, BLOCKING, card 4: the `agentfake` kit imports `sheddriver`, so the internal `sheddriver` tackler tests importing it form an import cycle — fixed: no kit; `sheddriver`'s `tackler_test.go` keeps a local `agentFake` per `PATTERN-testkit`'s cycle rule, and loom's `arbiterFake` stays, its `Start` retyped.
- criteria-coverage, MEDIUM, card 8: the seeding sites' recipe pick is untested — fixed for batten with `childDriverKind` and `TestChildDriverKind`; rejected for loom's site: `resolveSeedRecipe` returns `shedrun.RecipeLoom` for every unseeded run (`internal/loomcli/sharedbootstrap.go:184-189`), and a seeded run keeps its recorded driver (`resolveSeedDriver`, `sharedbootstrap.go:169-179`), so that site's pick can only read `loom.yaml`; card 8 states this.
- criteria-coverage, MEDIUM, cards 4 and 8: `goDriverDepsFor`'s choice of tackler settings and recipe line by `c.recipe` is untested — fixed: card 8 adds `tacklerFor` and `TestTacklerFor`, pinning the darn 0/4 pair and the recipe line.
- criteria-coverage, MEDIUM, card 13: batten's driver-lock path and stale files are untested — fixed: `battenDriveLock`, asserted in `TestBattenDriverDeps`; the adoption of a live loop stays with `shedverbs`, since `runUntilStop` follows a live loop generically (`internal/shedverbs/loopwait.go:94-95`) and batten only tells it its loop spec, which card 11's test pins.
- criteria-coverage, MEDIUM, card 13: no test asserts the tackler runner is built over the prime's reed geometry — rejected: `shuttleengine.NewRunner` keeps its anchor and worktree root in unexported fields with no accessor (`internal/shuttleengine/run.go:138`), so no test can read the geometry back without a new production accessor the record does not ask for; the card names the constructor and its geometry for review.
- criteria-coverage, LOW, card 12: the episode key's off-by-one is pinned only on synthetic markers — rejected: a halting row's persist appends the entry after `Call` returns (`internal/shedengine/run.go:358`), and card 12 states the rule.
- criteria-coverage, LOW, card 6: loom's seam closures (the arbiter outcome mapping, the verify closure) have no direct test — rejected: field-by-field copies over code the plan-level verify builds; the rule table's behaviour they feed is pinned in `TestGoDriver_Settle`.
- row-reachability, MEDIUM, card 8: a base-layer `batten.yaml` `driver_kind` may load without refusal, since `configengine.ValidateLocal` runs only for the local layer (`internal/configengine/layers.go:170`, `247-251`) — fixed: the battenshed row writes the key in the local layer, the layer `run_settings` writes.
- row-reachability, MEDIUM, card 13: eager config and runner loads in `driverDepsFor` would fail the run tests on a bare temp hub — fixed: those loads happen inside the `Tackler` closure on its first call.
- row-reachability, LOW, card 6: the unwired-seam rows need named harness fields, and the snapshot fakes record no call — fixed: `noArbiter`, `noVerify`, `noHandOver`, `noTamper` and `noCommit` fields, the snapshot fakes record `snapshot`.
- row-reachability, LOW, card 6: the harness's fake `Park` must write the marker — fixed.
- row-reachability, LOW, card 4: the goto rows need an input field and the fixture a `GotoRoutedFrom` default — fixed: `TacklerPromptInput.GotoRows`, fixture default true.
- test-economy, MEDIUM, card 11: the new spec test belongs in `TestSpecFor_FillsRunIdentityForSlug` — fixed: folded in, with no budget raise.
- test-economy, LOW, card 13: `TestBattenRunEnvelope` repeats the `TestRunCmd_*` assertions — fixed: dropped, with the failed and running-elsewhere cases as rows of the run tests.
- test-economy, LOW, card 7: `TestLoadConfig`'s refusal rows repeat `TestCheckOverride` — fixed: one wiring row kept.
- test-economy, LOW, card 7: the `TestModules_LocalLayerRefusals` shed row repeats the load test — fixed: dropped.
- test-economy, LOW, card 8: `TestRecipeOverride`'s inherit rows repeat card 7's resolver tests — fixed: trimmed to the recipe pick.
- test-economy, LOW, card 4: the three recipe rows each assert the whole stencil body — fixed: one row asserts the body, the others the line.
- forced-edits, LOW, card 8: the batten refusal-spec row and the `--child-driver` help still say "the task's driver_kind" — fixed in card 8, and `docs/overview.md`'s sentence in card 14.
- forced-edits, LOW, card 13: `Arm`'s doc names the deleted field — fixed: `internal/battencli#Arm` listed.
- forced-edits, LOW, card 6: an empty friction directory's report line is unspecified — fixed: it renders `none`, with a row.
- forced-edits, LOW, card 11: the step help says nothing of `--until-stop` — fixed: `battenVerbTexts` listed.
