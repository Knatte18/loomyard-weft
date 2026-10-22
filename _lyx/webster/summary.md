# Quarry v0.4.0 and C# glyph support under `language: csharp`

All five batches reached `status: done` on the first fork, with no recovery.
Every digest's `warnings` was null, and no digest carried `departures`.

## Batches

- **01-quarry-v040** (head `e6a03526b`): quarry raised to v0.4.0 and `Lang` set on every declaration.
  `planparser.ParseRef` became the one alphabet choice, and `language: csharp` is accepted.
  No scope-guard deviations.
- **03-planglyph-ref-alphabet** (head `5303ec951`): `GlyphLanguage` keeps only its bool.
  planglyph, plandag and the kick-start parse through `ParseRef`, the C# fixture and `TestGlyphChain_CSharp` landed, and `ScopeGuard` receives the plan.
  Scope-guard deviations, informational: `internal/planglyph/create.go`, `internal/planglyph/glyphchain_integration_test.go`, `internal/planglyph/glyphchain_test.go`, `internal/planglyph/handle.go`, `internal/planindex/planindex.go`.
- **05-go-only-checks** (head `d7fdbb19c`): Go-toolchain checks run only under `language: go`.
  Added the plan-language-mismatch and C# namespace Rename-pair refusals, cross-alphabet ambiguity with `.go`-only build-constraint partitioning, and C# Create handles bound with `Owner`.
  Scope-guard deviations, informational: `internal/planglyph/buildenv.go`, `internal/planglyph/csharphead.go`, `internal/planglyph/glyphchain_test.go`, `internal/planglyph/languagemismatch.go`.
- **10-csharp-rename-derivation** (head `17075ded7`): C# Rename declaration derivation, C# re-sign arrow acceptance and checks before the resolve, plan-wide rewrite of a recorded C# re-sign or refusal as not done, and the batcher boundary between a C# re-sign and a later use of its member.
  Scope-guard deviations, informational: `internal/batcher/boundary.go`, `internal/planglyph/csharphead.go`, `internal/planglyph/handle.go`, `internal/planglyph/resign.go`, `internal/planglyph/status_enforcement_test.go`, `internal/planindex/planindex.go`, `internal/websterengine/beginbatch_test.go`, `internal/websterengine/recordbatch.go`.
- **14-package-docs-pattern** (head `c4a162f51`): package docs, the `PATTERN-glyph-conversion-chokepoint` change, the plan spec, the planner stencil and the Plan-Review rubric.
  No scope-guard deviations.

## Scope guard

Deviations were reported for batches 03, 05 and 10 only, and each list is above.
None was treated as a reason to stop.

## Verify gate findings

The plan-level verify failed on attempt 1 of 3 with one failing identity: `github.com/Knatte18/loomyard/internal/loomcli.TestGateParity_PlanGate/VerifyNestedModule` (test).
The findings file noted that no recorded card touched a failing package.


## Verify gate fix commits

The verify-fix fork's commits above the pre-fix head, and the PATTERN departures their messages declare.

- `55eb851f903fbe69df50eef6cfcfd0104d7f2bf2` fix: parity fixture runs the nested-module verify row under language go
