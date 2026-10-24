```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the round-1 diff. Cases run, correct unless noted.

**Per function, the case run**
- `runBuild` (`gatecli/build.go`):
  - Exit 0: correct.
  - Non-zero exit: correct, sets `buildFailure` and `buildExit`.
  - Deadline hit: correct, forces exit 1 and writes the timeout line.
  - Shell not started: refusal, the call halts.
  - `e.ctx` cancelled: the build counts as failed with `halt` set. Acceptable.
  - The tier's `timeout_min` starts inside `runTier` after the build. Correct.
- `reportBlocked` and the `runTest` loop: build counted once before the first blocked tier. The call exits with the first failure's code. The blocked tiers print no tier summary line. Correct.
- `runListed`, `runGreenAware` and `planTierRuns`:
  - An empty scope (inert-only or empty diff) runs no build. Correct.
  - A `buildOnly` tier takes slots and `runTier` passes it with zero invocations. `emptyTierFailure` returns "" because no counts exist. Correct.
  - A `carry: false` + `shared_build` tier under an empty or no-test scope runs whole, with the build before it. Correct.
- `scopeInvocations`: explicit targets, `NoCarry`, `noTests` with and without `shared_build`, and an empty module list all correct.
- `filterArgument`: no terms, terms only, tier filter plus terms, and over-budget fallback correct. `QuoteArg` double-quotes on Windows, so `|`, `&` and `~` are safe there.
- `tierFingerprint`, `subtract`, `recordPass`, `carriesOverInert` and `recordInertCarried`:
  - New fields are `omitempty`, so existing fingerprints are unchanged.
  - A partial invocation is never recorded and never inert-carried.
  - A `NoCarry` tier keeps every invocation and records nothing.
  - A whole invocation is inert-carried only on a clean tree with an ancestor entry and an inert-only diff. Correct.
- `noneScope` and `allInertSince`: an empty diff, a read error and an empty `inert_paths` all leave the scope whole. Correct.
- `dotnetScope` and `assignShare`: whole, no-test-class, narrowed and empty-diff shares correct. Covered or dropped tiers are re-dropped by `planTierRuns`.
- `DeriveDiff`:
  - Deleted file seeds from its base version.
  - Untracked file seeds from the working tree.
  - Rename seeds by both paths.
  - A file outside `dir`, a non-`.cs` non-inert path, and no declared type all fall back whole.
  - Inert paths are dropped first.
  - An empty diff gives the empty scope. Correct.
- `DeriveCards`: inert target dropped, a target with neither file nor type falls back whole, a missing file with types seeds the types, and an empty seed gives the empty scope. Correct.
- `narrow`:
  - Closure, abstract-base selection of derived classes, inherited test classes, nested `+` terms, the more-than-one-csproj fallback, and no test class all correct.
  - A concrete test base does not select its derived classes, which matches the decision.
- `gateslot` config:
  - `decodeBuild` refuses a missing command or `timeout_min` and a `timeout_min` below 1. `shared_build` without a build is refused. `filter` without `{filter}` is refused.
  - `Inert` and `AllInert` are the same method values for `gatecli` and `dotnetimpact`.
  - A malformed `inert_paths` pattern matches nothing and is not refused. Safe direction, so no finding.
- `BuildSummaryLine`, `FailedTier`, `FindTierTimeout` and `verifytree.tierTimeoutDetail`: the build summary matches its pattern, and the timeout detail starts with "gate build ", which the landing verdict tests for. Correct.

**Findings**

1. MEDIUM, design, `internal/dotnetimpact/lex.go:218` (`stringBody`, evidence lines 213–222 and `rawBody` at 241): a run of three or more quotes is always treated as a raw string, even when the `@` prefix makes it verbatim.
   - A verbatim string that opens with an escaped quote, such as `@""""` or `@"""a"" b"`, is wrongly read as a raw string.
   - The scan then runs to the next `"""` or to the end of the file. Every identifier after that point is dropped from `Identifiers` and the type declarations.
   - This silently narrows to too few test classes, which the focus directive names as the failure to avoid. The fix is to take the raw path only when `!verbatim`.

2. LOW, design, `internal/dotnetimpact/source.go:262` (`startsTypeDecl` and `declaredName`): a `record` identifier followed by another identifier is read as a record declaration.
   - Examples are `foreach (var record in rs)` and `record is Foo`.
   - It adds a bogus type named `in` or `is` and skips tokens to the next `{` or `;`.
   - That bogus type enters the closure when its file changes. `Identifiers` includes keywords, so almost every file names `in` or `is` and every test class is then selected.
   - This over-selects, which is the safe direction, but it defeats the narrowing.
   - The fix is to require the `record` keyword to be followed by an identifier and then `(`, `<`, `:`, `{` or `;`, or to exclude `in`, `is`, `as` and `when`.

Not verified: `landingshed/verifygate.go:99` still calls `traceFailure` on a build-failure log. The effect on `Untraced` is unknown. I did not read `traceFailure`, so I raise no finding.

**Tool calls**
1. Read `dotnetimpact/lex.go` and `source.go`, which were absent from my context and are needed to run the lexer and parser edge cases.
2. Read `verifyrun.QuoteArg`, to check Windows quoting of `|`, `&` and `~` in a composed filter.
3. Grepped `FindTierTimeout` and `gate build ` callers and read the start of `runTier`, to check the build timeout detail string and the zero-invocation behaviour.
4. Read `emptyTierFailure` and `verifytree.tierTimeoutDetail`, to check that a no-invocation or vstest run is not failed as an empty tier, and to confirm the timeout detail wording.
