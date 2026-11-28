```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the review's findings and check the fix covers every member.

**Findings left unchecked:** none.

- **F1** (sites naming the batch key): covered. The only non-test literal naming the gate key is `BatchGateKey` itself; `stopKindBatch` and the struct tags are not the gate key.
- **F3** (sites that treat a tier as narrowed): covered in code. `ownPackageTiers` alone decides which tiers get their own share, so the notice, `listedPackages` and the guard step all follow it. One prose gap is below.
- **F5, F6** (statements that give the batch gate the dependents): covered. The remaining hit, `contracts/stencils/webster/webster-body-implementer.md:73-74`, describes the card gate and still holds.
- **F7** (tooling names in the change description): no other instance in `_lym/landing/summary.md`.
- **F2, F4**: coverage gaps, not sets.

**Finding**

- **LOW**
  - **Changed text:** the F3 exception added to the `ownPackageTiers` doc comment (`internal/gatecli/scope.go`).
  - **Problem:** its sibling statements still say every tier above tier0 that does not cover tier0 runs the cards' packages alone. A `carry: false` tier, any tier of a `--no-carry` call, and a tier whose command takes no targets now are not narrowed and not named on stderr.
  - **Evidence:** the `computeScope` doc comment reads "each listed tier above tier0 that does not cover tier0 runs the cards' packages alone … request.stderr names those tiers". `internal/gatecli/doc.go:154-156` reads "Each other listed tier runs the cards' packages alone … and the call names it on stderr". The same unqualified rule is in `loom-plan-spec.md:347`, `template.yaml:18` and the `--card` help. `doc.go` states the general `carry: false` whole-run rule elsewhere, so the nearest twins to correct are `computeScope` and the `doc.go` paragraph.

**Tool calls**

1. `git grep` for batch key literals and gate-call arguments: my context's search covered only `internal/`, and predates the fix.
2. `git grep` for scope wording across `.md`, `.go` and `.yaml`: my context's search did not cover the stencils.
3. `grep` of the webster and weaver stencils for gate and dependents wording: only line numbers and truncated hits were known.
