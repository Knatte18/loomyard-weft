```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens on the diff, which changes only `internal/loomcli/prfindings_integration_test.go`.

**Each behaviour and the test that pins it:**
- **`pr-findings` on a darn run wires the landing seams before building the shed.** Pinned by the `TestPRFindingsCLI_SendsBackBlockingFindings/Darn` row.
  - The darn recipe has a PR-Review row with `on_reroute: Code-Bouncer` (`contracts/recipes/darn-recipe.yaml:285-288`) and a Finalize row (`:295`).
  - So `prReviewSendBackEdge` (`internal/loomcli/prfindings.go:290-305`) reaches `wireLanding` and `newShed`.
  - If the wiring were dropped, `landingshed.NewFinalize` would refuse with `Deps.OpenFabric must not be nil` (`internal/landingshed/finalize.go:107`). The edge would then return an error, the exit code would be non-zero, and the row's `t.Fatalf` would fail it.
- **`pr-findings` on a tapestry run wires the landing seams (Parent-Align).** Pinned by the `TestPRFindingsCLI_SendsBackBlockingFindings/Tapestry` row.
  - Without the wiring, `NewParentAlign` refuses (`internal/landingshed/parentalign.go:85`), so this row would fail the same way.
- **A BLOCKING findings set goes back to Code-Review (`send_back: true`).** Pinned by the `send_back` assertion in both rows (`prfindings_integration_test.go:72-74`).
- **The seedless tapestry default.** The base test ran with no seed, so it relied on `resolveRecipe` falling back to tapestry when a run has no seed (`internal/loomcli/arm.go:105-115`). The rewritten test seeds every row, so it no longer exercises that fallback.
  - Other loomcli tests cover the fallback: `grep` finds `RecipeTapestry` / `resolveSeedRecipe` in `arm_seed_test.go`, `cli_test.go` and `peers_test.go`, among others.
  - No coverage gap.

The reach and mutation reports show that the diff changes no production function, so no production behaviour is left uncovered.

**Findings:** none.

**Tool calls:**
1. `grep` of `internal/landingshed/finalize.go` and `parentalign.go` for their nil-dependency refusals, and of `internal/loomcli/*_test.go` for tests of the seedless tapestry default. My context showed neither whether Finalize refuses nil seams (which decides whether the darn row can fail) nor whether another test covers the default the base test used to exercise.
