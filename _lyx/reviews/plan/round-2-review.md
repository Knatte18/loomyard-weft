---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: _lyx/plan/07-filter-placeholder.md
    summary: Card 7 prescribes a refusal for a tier filter holding a percent sign under cmd, but no named test reaches it on a non-Windows host
    fix: "In 07-filter-placeholder.md add a goos string first parameter to the Create line for filterArgument, so it reads func filterArgument(goos string, number int, tier gateslot.Tier, terms []string) (string, error), and say in the Intent that renderTierCommand passes runtime.GOOS and filterArgument quotes through verifyrun.QuoteArg(goos, ...). Add a Create line for plan:internal/gatecli#TestFilterArgument -> func TestFilterArgument(t *testing.T), and to the Tests section a sentence saying TestFilterArgument is a tier0 table over goos in which a tier filter holding a percent sign is refused under windows with the exact refusal text naming the tier number and the lyx config gate way forward, and is quoted under linux. Checked with lyx loom validate-plan --overlay against the baseline run of {}, both report ok true with the same informational findings."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/08-dotnet-scoper.md
    summary: Card 8's C# seed tree names CircleTests and OtherTests only as classes naming a type, never as test classes, so no narrowed row selects a class under the Decision 2 test class rule
    fix: "In 08-dotnet-scoper.md, in the TestGateTest_Scopes paragraph that lists cs/Lib.Tests/CircleTests.cs and cs/Lib.Tests/OtherTests.cs, state that each class carries the TestFixture attribute and one method carrying the Test attribute, that CircleTests' method body or fields name Circle, and that OtherTests names neither Circle nor Unused. No target changes."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/06-derive-entry-points.md
    summary: Card 6 bundles the two new exported impactset reads, ChangedPaths and AncestorBase, into the dotnetimpact entry-point card although they are public surface reused across packages
    fix: "Insert a card with lyx loom insert-card before card 6 (slug impactset-diff-reads) that holds the Create lines for plan:internal/impactset#ChangedPaths and plan:internal/impactset#AncestorBase, an Extends line for internal/impactset/doc.go, the Intent sentence about the two exports and the ImpactSummary sentence about the two new wrappers; then remove those two Create lines, the impactset doc.go Extends line and that Intent paragraph from the card now numbered 7, and add the two symbols to its Uses. Not overlaid, since insert-card renumbers; no admissible fix needs a check waiver."
---

### [LOW] Percent-sign refusal in card 7 has no reaching test

**Location:** `_lyx/plan/07-filter-placeholder.md`, the paragraph on a quoting error of the tier's own `Filter`.
**Issue:** The card prescribes a refusal naming the tier and the way forward `fix the tier's filter with "lyx config gate" from the prime`, reached by a tier `filter` holding `%` under Windows' `cmd`.
`verifyrun.QuoteArg(goos, value)` returns `ErrPercentUnderCmd` only for `goos == "windows"`, and the card's `filterArgument` is not given a `goos`; it would read `runtime.GOOS`, as `quoteAll` does in `internal/gatecli/tiers.go`.
The named tests are rows of `TestRenderTierCommand`, which call `renderTierCommand` on the host's OS, so no row on a Linux host reaches the branch, and a refusal no test reaches is the rubric's reachability finding.
No upstream check rejects the input: `decodeTier` refuses only a `filter` on a tier without `{filter}`.
The fix is a `goos` parameter and one small table, checked with `lyx loom validate-plan --overlay`.

### [LOW] Card 8's seed test classes are not stated to be test classes

**Location:** `_lyx/plan/08-dotnet-scoper.md`, the `TestGateTest_Scopes` paragraph listing the C# seed tree.
**Issue:** The tree is listed as `CircleTests.cs` "with a `CircleTests` class naming `Circle`" and `OtherTests.cs` "with an `OtherTests` class".
Decision 2 and card 5 make a class a test class only when it carries a test attribute or holds a method that carries one, so a class built to the card's words selects nothing: the narrowed rows then reach `NoTestClass` instead of the project and term they assert, and `OtherTests` could never be the unrelated test class left out.
Card 11's `Unused.cs` row depends on the same tree and is unaffected, since it needs a type no test class names.
The fix states the attributes and what each class names.

### [LOW] Card 6 bundles two exported impactset reads with the dotnetimpact entry points

**Location:** `_lyx/plan/06-derive-entry-points.md`, the `Create` lines for `ChangedPaths` and `AncestorBase`.
**Issue:** `ChangedPaths` and `AncestorBase` are new exported members of `internal/impactset`, called from `internal/dotnetimpact` in this card and from `internal/gatecli` in card 10.
The rubric's granularity rule gives a symbol that is part of a module's public surface, or reused across packages, its own card even when one card is its only consumer; here they sit beside two entry points, a tree scan and two test tables of another package.
The two are thin wrappers over unchanged functions, so the cost of splitting is small and the review of card 6 then covers `dotnetimpact` alone.

## Focus verdicts

1. Card 10's empty share for an inert-only base group is not a defect.
   Under `go` the card names the closing switch's result for a call whose groups are all inert-only (per-tier shares, `guardStep` set, `guardDir` the `tier0` dir, `guards` nil, as the `anyWhole` branch returns), and says the group counts as narrowed and leaves the guard tests to the module scan "as a whole tier does", which carries the mixed case into the same branch.
   Under `none` it gives the per-group share, every other group and baseless tier staying whole, and `guardStep` false, so `planTierRuns`' `--guards` refusal is not met.
   I read `computeScope`'s closing switch in `internal/gatecli/scope.go` against this: the `!narrowed` return and the `guardsByTier(nil)` return are the two the card rules out by name.
2. Card 8's seed tree meets both named conditions.
   A `.csproj` sits above every seed file (`cs/Lib/Lib.csproj` above `Circle.cs` and `Unused.cs`, `cs/Lib.Tests/Lib.Tests.csproj` above the test files), and `Unused` is a production type no test class names, so the no-test-class row in card 11 reaches `NoTestClass`.
   The remaining gap is F2: the test classes are not stated to carry test attributes.
   The tier `dir` is `.` in `scopesConfigWith`, so the project path `cs/Lib.Tests/Lib.Tests.csproj` the rows assert is the path relative to the `dir`.
3. Card 11's `runBuild` refusal is not a defect.
   The card refuses a build whose process is nil while `ctx` is not done, naming the build's `dir` and the way forward, and carries it in `tierResult.refusal`, as `runInvocations` does at `internal/gatecli/tierrun.go` for a tier.
   A missing `build.dir` reaches it, no upstream check rejects that `dir` (`decodeConfig` in card 1 validates keys, not existence), and the scenario row with a missing directory is the named test.
   The absent-shell case shares the same branch and needs no second row.

## Rejected

- Card 9's and card 7's "one production caller" and "three production callers" tallies match the tree: `renderTierCommand` is called only by `runInvocations`, and `tierFingerprint` only by `diffBaseGroups`, `greenRun.subtract` and `greenRun.recordPass`.
- Card 6's git-failure fallback and its cancelled-context check have no reaching row, which the card says openly; a local git read failing over a valid repository is a transient I/O error, which the rubric exempts.
- The Windows percent-sign refusal on targets and args in the existing `quoteAll` has no `gatecli` test either, but it predates the plan and is not a finding against it.
