```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fixer added or rewrote holds against the code at HEAD.

no findings

Each added claim matches what my context already shows:
- **Card 5:** `TestBuild_EveryRegisteredEngineBuilds` builds a one-row recipe for each name in `shedrecipe.Names()` and sets no `BlockGate` (`internal/shedbuild/build_engines_test.go:106-113`). Card 2 defines `BlockGateAlways`, so setting it is a step that can be carried out.
- **Card 3:** `contracts/specs/shed-recipe-spec.md` does have a "Document shape" paragraph and a row-key sentence (lines 87-91).
- **Card 6:** Card 2, line 31, says `blockProducer.Call` returns Done at once, which supports the Reinvoke rationale. `InterruptPolicies` is in `internal/loomshed/interruptpolicy.go`, beside where `NameMotif` is created. `TestInterruptPolicies_MatchAssembledRows` builds tapestry, darn and sampler today (line 25), so "adds `shedrun.RecipeArras`" is a real change.
- **Card 7, battenshed:** `internal/battenshed/doc.go:137` reads "a tapestry, darn or sampler fixer's escalate halt", as the new Intent line says.
- **Card 7, `docs/overview.md`:** each sentence the card names exists:
  - the loomrecipe line (262)
  - the shedcli arming-table line (267)
  - the `lym loom` verbs list (537)
  - the `EscalateStopPrefix` escalate-halt sentence (612)
  - the batten bullet's child recipes (681)
  - the `Run-Shed` row's seeded runs (789)

No further tool calls made.
