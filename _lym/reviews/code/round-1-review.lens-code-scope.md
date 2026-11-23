```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped each change in the diff to a goal point of the board entry, and checked whether every goal point has a change that builds it.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries, so the four goal points under `## Goal` are the whole list:
- **G1:** select the changed tests by test project and fully qualified name (`FullyQualifiedName~`) from the changed `*.cs` files under test projects.
- **G2:** build and run them on the merge base in a scratch checkout through the hub's own tier configuration, red judged on the base and green at the head as for Go.
- **G3:** a test that does not compile on the base, because it reads a member the change adds, is red with that reason, not a missing seam.
- **G4:** halt only when selection or build fails for a reason the writer can act on.

**Per change: the decision it traces to**
1. `internal/dotnetimpact/changedtests.go` (`ChangedTests`, `ChangedTest.FilterTerm`/`Label`, `qualifiedName`) and `source.go` (`TestMethods`, `testMethod`, `methodEnd`, `statementEnd`, `matchingBackward`, `endMember`, `inTypeBody`), with their tests and the `doc.go` paragraph → **G1**.
   The `narrow.go` `filterTerm` switch to `qualifiedName` is forced by change 1: both now share one spelling of the qualified name.
2. `internal/gitrepo/blobread.go` `WriteTreeAtRevision` and `writeBlobFile`, and `internal/fabricengine/codereads.go` `ExportCodeTreeAt`, with their integration-test rows → **G2** (the scratch checkout).
3. `internal/gatecli/redfirstdotnet.go`:
   - `run`, `changedTests`, `byProject`, `nearestProject`, `filterTerms`, `dotnetLabels` → **G1**
   - `keepHeadProjectFiles`, `targetTiers` and the tier run → **G2**
   - `judgeDotnetRedFirst`, `projectCompileError`, `csharpCompileError` and the `build_errors` key → **G3**
   - the `redFirstCannotJudge` exits for abstract classes, no or two `.csproj`, and no `{targets}` tier → **G4**
4. `testRequest.terms` and `checkoutRoot` (`cli.go`), the `checkoutRoot` swap in `run.go:108-110`, and the terms/partial change in `tiers.go` `scopeInvocations`, with the `tiers_test.go` row → **G2** (run through the hub's tiers in the checkout).
5. `gateslot.TierFailureNames` (`runlog.go`) with `TestTierFailureNames` → **G2** (reading red from VSTest failure names).
6. The `redfirst.go` dispatch on `cfg.Scoper == "dotnet"` and the skip of `redundancyModule` for dotnet → **G1** and **G2**.
   The extraction of `redFirstNoTest` and `redFirstVerdictExit` is forced by change 3, which shares the Go path's verdict "as for Go".
7. Docs and stencil: `gatecli/doc.go`, the `redfirst.go` Long text and comments, the `gateslot/redfirst.go` `RedFirstUnjudgedExit` doc, `loomshed/doc.go`, the `loomshed/redfirstgate.go` doc and log message, `shedrecipe/doc.go`, `contracts/recipes/darn-recipe.yaml`, `contracts/stencils/darn/darn-template-write.md` and `docs/overview.md` → forced by changes 3 and 6.
   They described the old "scoper is not go" behaviour, which the change makes stale.
8. `dotnetfixture_integration_test.go`, which splits `dotnetFixtureFiles` out of `writeDotnetFixture`, and `redfirstdotnet_integration_test.go` → forced by changes 3 and 6 (test of the new path).
9. Commits `fe1e4116e`, `be52b316b`, `875efcd05` and `4ffca3704` (comment line-break and header rewording) → forced by the comment lint on lines this diff wrote.

No change is unrequested scope, and no commit in the range carries a `Pattern-Departure:` trailer.

**Per decision: the changes that build it**
- **G1:** changes 1, 3 and 6.
- **G2:** changes 2, 3, 4, 5 and 6. "Green at the head as for Go" matches the Go path, which also runs only on the base; the head's green comes from the row's other gates.
- **G3:** change 3 (`judgeDotnetRedFirst` and `projectCompileError`), pinned by the `ATestThatReadsAMemberTheChangeAddsIsRedSinceItDoesNotBuildOnTheBase` integration row.
- **G4:** built only in part; see the finding.

**Findings**

1. **MEDIUM · scope** · `internal/gatecli/redfirstdotnet.go:135` and `internal/gatecli/redfirst.go:214-215`
   - **Summary:** G4 is built only in part. A dotnet tier run that fails outside the test project exits 1, which the red-first gate treats as a terminal failure. That includes a failed `dotnet restore` or base production code that does not build, and neither is something the writer can act on. The entry asks to halt only when selection or build fails for a reason the writer can act on, but nothing tells these cases apart from an actionable build failure.
   - **Evidence:**
     - `redfirstdotnet.go:135-137` keeps the tier's exit code as `unjudgedCode`.
     - `redfirst.go:214-215`: `redFirstVerdictExit` returns `output.Err`, exit 1, "judged none".
     - `internal/loomshed/doc.go:66-14` (diff 010): "any other exit code, a timeout or a runner crash fails terminally".
     - `_lym/landing/summary.md:39-40` says so explicitly: "A build failure outside the test project, such as base production code that does not compile or a failed restore, judged none … exits 1 … which the gate treats as terminal".
     - The `ARunThatJudgedNoneExitsOne` row in `redfirstdotnet_test.go` pins `restore failed` → exit 1.
   - **Fix direction:** a restore or base-build failure outside the test project should take an exit the gate treats as not halting the writer: unjudged, or a retry. The alternative is a decision recorded with the parent that keeps the Go path's terminal treatment.

No tool calls made; everything came from the inherited context.
