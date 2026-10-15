# Card 14 — PATTERN-model-choice-mapping and its scan

**Create:**
- `plan:internal/shuttleengine#modelChoiceMappingViolations` -> `func modelChoiceMappingViolations(t *testing.T, base string) ([]string, int)`
- `plan:internal/shuttleengine#TestModelChoiceMappingRule` -> `func TestModelChoiceMappingRule(t *testing.T)`
- `pattern/PATTERN-model-choice-mapping.md`

**Edit:**
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D2's guard, landing with its PATTERN entry once cards 1 to 10 have moved every mapping onto `shuttleengine.NewModelChoice`.
The rule: outside `internal/modelspec` and `NewModelChoice`, no production code reads any `modelspec.Resolved` field (`Engine`, `Model` or `Params`); every function that resolves a model key returns a `shuttleengine.ModelChoice`, or a list of them, built by `NewModelChoice`, a timeout it also resolves staying beside the choice.

`modelChoiceMappingViolations` walks the production Go files under `base` with `scankit.Walk`, skipping `internal/modelspec/`, and parses each file that imports `internal/modelspec`.
In such a file it collects the identifiers that hold a `modelspec.Resolved`: the left-hand identifiers of an assignment or `var` whose right-hand side is a call of a method named `Resolve` (the registry's `reg.Resolve(spec)`), and every identifier declared with the type `modelspec.Resolved` (a `var`, a parameter, a result, a range value over a map or slice of it).
A selector `x.Engine`, `x.Model` or `x.Params` whose `x` is such an identifier is a violation, reported as the file and line, except inside the body of `NewModelChoice` in `internal/shuttleengine/modelchoice.go`.
The scan is syntactic, so it cannot see a `Resolved` reached through a struct field or a call chain; `PATTERN-model-choice-mapping.md` names that bound, and review covers the rest.

`TestModelChoiceMappingRule` runs it over synthetic trees and the real module:
rows where a file reads `resolved.Model`, `resolved.Params["effort"]` and `resolved.Engine` after `resolved, err := reg.Resolve(spec)` each report one violation; a parameter `r modelspec.Resolved` read as `r.Model` reports one; a `var r modelspec.Resolved` read as `r.Engine` reports one; a range value over a `map[Role]modelspec.Resolved` read as `v.Params["effort"]` reports one; a file passing `resolved` whole to `shuttleengine.NewModelChoice` reports none; the same read under `internal/modelspec/` and inside `NewModelChoice` reports none;
the `RealModule` row requires a floor of scanned files and no violation.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/shuttleengine` row rises by one.

`PATTERN.md` gains, under `## Agents and prompts` after `PATTERN-shuttle-provider-seam`, the entry

```markdown
- `PATTERN-model-choice-mapping` — Mapping a resolved model-spec onto a spawn: only `shuttleengine.NewModelChoice` reads a `modelspec.Resolved` field. (test) — [background](pattern/PATTERN-model-choice-mapping.md)
```

and `pattern/PATTERN-model-choice-mapping.md` holds the full rule, the resolvers it binds, the timeout-beside rule, the scan and its bound.
`PATTERN.md` stands two bytes under its headroom line, so the same commit shortens other entries by at least the new line's length, each only where its background file already states what is cut, until `lyx loom lint-pattern` reports no growth past the headroom.

**ImpactSummary:** one new scan test and one new PATTERN entry with its background; other PATTERN entries lose only words their backgrounds already carry, and no production code changes.

**Verify:** lyx loom lint-pattern

**Commit:** 14: PATTERN-model-choice-mapping and its scan
