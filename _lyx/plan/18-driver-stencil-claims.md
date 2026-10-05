# Card 18 — Driver stencil checks join the claims table and the recipe-blind scan

**Edit:**
- `contracts/stencils/claims_test.go`
- `cmd/lyx/driverstencil_test.go`

**Delete:**
- `internal/loomcli/driverstencil_test.go`

**Uses:**
- `internal/shedrun#ParkMarkerFileName`
- `internal/loomshed#InterruptPolicies`

**Intent:**
Finding F2: `internal/loomcli/driverstencil_test.go` holds five tests, each pinning one phrase of the shed driver stencil, which `PATTERN-test-economy` rules out.

- The park-marker, files-nothing, `friction: failed` and `parent_notice` checks become claims in a new `shed-template-driver.md` row of `wordingClaims` in `contracts/stencils/claims_test.go`.
  The park-marker claim is built from `shedrun.ParkMarkerFileName`, so a rename of either side still fails; `lyx selfreport create` is a mustNot claim.
- The no-loom-row check is a recipe-blindness scan of the same stencil, so it becomes extra forbidden tokens in `TestDriverStencil_IsRecipeBlind` in `cmd/lyx/driverstencil_test.go`: every row name in `loomshed.InterruptPolicies`, matched as a whole word with the boundary the deleted test used, so a hyphenated neighbour does not match.
- `internal/loomcli/driverstencil_test.go` is deleted.

Each moved check keeps its why as the claim's `why` or the token's label.

**ImpactSummary:** test-only: one test file deleted, its checks moved into the claims table and the existing recipe-blind scan.

**Commit:** 18: stencils: driver stencil checks join the claims table and the recipe-blind scan
