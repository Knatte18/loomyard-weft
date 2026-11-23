```yaml
lens: code-affected-set
outcome: reported
```

Lens task: code-affected-set. For each change the diff makes, find everything outside the touched lines that it forces, and report what the diff leaves wrong.

**Per change: the affected set I found, and whether it is covered**

1. **Red-first under the dotnet scoper** (`runRedFirst` in `internal/gatecli/redfirst.go`, and `redfirstdotnet.go`).
   - Set: cobra `Long` help, the `runRedFirst` doc, `gatecli/doc.go` red-first section, `gateslot.RedFirstUnjudgedExit` doc, `loomshed/doc.go`, `loomshed/redfirstgate.go` doc and log line, `shedrecipe/doc.go`, `contracts/recipes/darn-recipe.yaml` comment, `contracts/stencils/darn/darn-template-write.md`, `docs/overview.md` (two places), the shared `noteScoper` refusal text.
   - Covered: all of them except `noteScoper`, the `gatecli/doc.go` envelope-label line and the `docs/overview.md` unjudged list (findings 2–4).
2. **New `cfg.Scoper` literal branches** in `redfirst.go`.
   - Set: PATTERN-code-conventions, "No hard-coded choice". No scoper constant or table exists anywhere in `internal/`.
   - Not covered (finding 1).
3. **New `testRequest.terms` and `checkoutRoot` fields** (`cli.go`, `scopeInvocations` in `tiers.go`, `runTest` in `run.go`).
   - Set: `scopeInvocations` doc, `TestPlanTierRuns` row, `gatecli/doc.go`.
   - Covered. Downstream readers of `request.root` after the swap (`runBuild`, `untrackedBeforeRun`, `touchedFiles`, `writeRecord` Worktree, `tierFailedRun`) belong to the correctness lens.
4. **`redFirstNoTest` and `redFirstVerdictExit` extracted** from `redFirstCall.run`.
   - Set: the Go path's callers and the existing `redfirst_test.go` rows.
   - Covered; message text unchanged.
5. **`gateslot.TierFailureNames` added.**
   - Set: runlog siblings, the `gatecli` caller, `TestTierFailureNames`.
   - Covered.
6. **`gitrepo.WriteTreeAtRevision` and `fabricengine.ExportCodeTreeAt` added.**
   - Set: PATTERN-gitrepo-client-boundary (go-git reads only through `readGoGit`) and PATTERN-fabric-boundary (git only through fabric, through a general seam).
   - Covered: the read goes through `readGoGit`, and fabric exposes a general path-based seam. The `blobread.go` file header and the codereads test header are updated.
7. **`dotnetimpact` changes**: `qualifiedName` extracted from `filterTerm`, `TestMethods` and `ChangedTests` added.
   - Set: `doc.go` paragraph, `TestParseSource` rows, `narrow.go` caller.
   - Covered.
8. **`writeDotnetFixture` split into `dotnetFixtureFiles`.**
   - Set: existing callers keep the same signature; the new caller is `TestGateRedFirst_Dotnet`.
   - Covered.
9. **Commit messages in the range.**
   - No `Root-Cause:`, `Loosened-Check:` or `Pattern-Departure:` trailers. The entry asks for a feature, not a bug fix, so no `Root-Cause:` is required.
   - No `Pattern-Departure:` covers finding 1.
10. **Declared PATTERN changes**: none are declared in the directive. Nothing to land.

**Findings**

1. **BLOCKING · design · `internal/gatecli/redfirst.go:135,148,166`**
   - Summary: the diff adds three new branches on the language literal `cfg.Scoper == "dotnet"` / `!= "dotnet"`.
   - Rule: PATTERN-code-conventions, "No hard-coded choice", says no code path branches on a language literal outside a table keyed by name. It also says a change that touches such debt "never adds another".
   - No `Pattern-Departure:` trailer excuses it.
   - Evidence: `pattern/PATTERN-code-conventions.md:15-28`, and the diff hunks at `redfirst.go:135`, `148` and `166`. A grep found no scoper constant or table in `internal/`, while 8 literal sites already exist.
2. **LOW · consistency · `internal/gatecli/redfirst.go:136`**
   - Summary: under the new condition (scoper neither go nor dotnet), red-first still prints `noteScoper`, "the hub's scoper is %s, not go, so the step judged nothing". That text is shared with the redundancy step and no longer states why red-first passes: dotnet is now judged too.
   - Evidence: `internal/gatecli/redundancy.go:40`, `redfirst.go:135-137`.
3. **LOW · consistency · `internal/gatecli/doc.go:60` (diff part 1, the exit-contract list)**
   - Summary: the exit contract still says "the envelope lists the red tests as `&lt;package&gt;.&lt;Name&gt;`". A dotnet red run lists `&lt;Class&gt;.&lt;Method&gt;` (`dotnetLabels`).
   - Evidence: `redfirstdotnet.go` `dotnetLabels`; doc lines in diff 006, around doc.go 500.
4. **NIT · consistency · `docs/overview.md:997-999`**
   - Summary: the unjudged-exit list says "C# tests no filter or tier can reach". It leaves out the other unjudged case the code has, a test file whose `.csproj` cannot be told (none, or two in its directory). `gatecli/doc.go` and the summary name that case.
   - Evidence: diff 003, lines 10-12; `nearestProject` and `byProject` in `redfirstdotnet.go`.

**Tool calls**
- grep across `internal`, `docs` and `contracts` for leftover "scoper is not go" wording and the `noteScoper` definition: my context lacked the docs outside the diff and the definition of the shared note.
- grep for a scoper-name constant and a count of the literal `"dotnet"` scoper branches: my context lacked whether a table or constant exists, which finding 1 depends on.
