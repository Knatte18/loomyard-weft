```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens over the plan's guards, refusals, branches and test rows.

**Per guard: reachable or not**

- Card 1: the `scoper` refusal (`want go, none or dotnet`) is reachable from hub-file input.
- Card 1: a `filter` without `{filter}` is reachable. `tiers` is an open map, so configengine passes the key through to `decodeTier` (`internal/gateslot/config.go:56-58`).
- Card 1: `build` without `command`, without `timeout_min`, and with `timeout_min` below 1 are all reachable once `build` is an open map, which card 1 makes it.
- Card 1: `shared_build: true` with an empty `build` is reachable, since the template fills `build: {}`.
- Card 1: `Config.AllInert` being false for an empty list or an empty path set is reachable.
- Card 2: `FailedTier`'s build branch is reachable once card 11 prints the build line.
- Card 2: `recordFailure`'s build branch is reachable for a failed run that did not time out.
- Card 2: `verdict`'s build-timeout reason is reachable. `tierTimeoutDetail` cuts on `"build exceeded timeout_min "`, which the planned line holds (`internal/verifytree/verify.go:477-490`).
- Card 2: `FindTierTimeout` returning `build` is reachable.
- Card 2: the `TestVerify_Scenario` and `TestMergeIn_PlanVerifyFailureRecord` rows are reachable.
- Card 3: the empty seed for a directory ref or a namespace self glyph is reachable.
- Card 5: in `narrow`, a seed file with no `.csproj` above it, a selected class with none, a nearest directory holding two `.csproj` files, and `NoTestClass` are all reachable from `TestNarrow`'s inputs.
- Card 6, `DeriveDiff`: the "base is empty" fallback is not reachable from gatecli. Card 8's `dotnetScope` runs a tier with no base whole without calling it, and `diffBaseGroups` never yields a group with an empty base (`scope.go` `diffBaseGroups`). It is reachable only from the test row.
- Card 6, `DeriveDiff`: a non-ancestor base, a non-`.cs` path that is not inert, a `.cs` change outside `dir`, a typeless file, and an inert-only or empty diff are all reachable.
- Card 6, `DeriveDiff`: the git-read-failure fallback has no reaching test, and the card's two paragraphs disagree on what it is.
- Card 6, `DeriveCards`: all of its fallbacks are reachable, as are the `bin/` and `obj/` skip rows.
- Card 7: the over-budget fallback is reachable. The quoting error is reachable only on Windows, as a propagated error.
- Card 8: the whole-fallback line, the empty share, the `--guards` refusal under `dotnet`, the `--card` unknown-id refusal under `dotnet`, and the cancelled-context error are all reachable.
- Card 8: the `noTests` share and the narrowed `TestGateTest_Scopes` rows cannot be reached with the fixture tree as written (F2).
- Card 9: the `recordPass` `NoCarry` skip and the `scopeInvocations` `NoCarry`-whole branch are reachable.
- Card 9: the `subtract` "`NoCarry` tier with a whole entry keeps its invocation" guard is not reachable from a real entry point (F3).
- Card 10, narrowed gates: the inert empty share under `go` and `none` is overwritten by `computeScope`'s existing `!narrowed` case when every group is inert (F1).
- Card 10, narrowed gates: the untracked non-inert file and "an empty diff is not inert-only" are reachable.
- Card 10, whole gates: the inert carry, and its negative rows for a non-inert commit, a dirty tree, `carry: false` and a changed list, are all reachable.
- Card 11: the `buildOnly` run, the build failure blocking later opted-in tiers, a tier that is not opted in still running, the build timeout, the carried tier with no build, and the empty-diff `carry: false` opted-in tier are all reachable.
- Card 11: the `TestGateTest_Scopes` "type no test class names" row cannot be reached as written (F2).
- Card 12: the skips for a missing `dotnet`, a failing SDK and an empty offline restore are all reachable.

**Findings**

F1. MEDIUM, design, card 10 (`10-inert-paths.md`, the narrowed-gates bullet). The inert empty share is lost when every base group is inert.
Card 10 gives an inert-only base group the empty share and skips `impactset.DeriveTargets` for it, but it never says that such a group counts as narrowed.
The existing switch at the end of `computeScope` returns `scope{whole: true}` when `!narrowed` (`internal/gatecli/scope.go`, `switch { case !narrowed: return scope{form: form, whole: true, ...}`).
Under `go`, a diff that is all inert would then run every listed tier whole, which defeats the rule card 10 adds and its own `docs/note.md` row.
`none` has the same problem if it reuses that switch.
The card should say that an inert group's empty share counts as a narrowed result, keeping the guard step over tier0's module scan.

F2. MEDIUM, consistency, card 8 (`08-dotnet-scoper.md`, the `TestGateTest_Scopes` C# tree) and card 11 (its `dotnet` `TestGateTest_Scopes` rows). The fixture tree cannot reach the narrowed state the rows assert.
The tree is `cs/Lib/Circle.cs` beside only `cs/Lib.Tests/Lib.Tests.csproj`, so no `.csproj` sits above `cs/Lib/Circle.cs` up to the tier's `dir`.
Card 5's `narrow` ("Every seed file must have a `.csproj` in some ancestor directory up to the `dir` … else `Whole`") and Decision 3 send that change whole.
So the computed, `--base` and `--card` rows can never run `Lib.Tests.csproj` with the `CircleTests` term.
Card 11's "edit to a type no test class names" row can never reach `noTests`, and so never reaches `buildOnly`.
The tree also holds no production type that no test class names.
The fix is to add a `cs/Lib/Lib.csproj` and an unnamed production type to the seed tree.

F3. LOW, design, card 9 (`09-fingerprint-and-carry.md`, the `subtract` bullet and its `TestGreenRecord` row). The guard that keeps a `NoCarry` tier's invocation over a whole entry guards a state that cannot occur.
`recordPass` writes no entry for a `NoCarry` tier, and card 9 itself writes that rule.
`NoCarry` joins the fingerprint, so an entry written before the tier was set `carry: false` sits under a different fingerprint.
Card 10's inert carry also excludes `NoCarry` tiers.
So `covered` can never report whole for a `NoCarry` tier's current fingerprint, and only the hand-built record in the test row reaches the guard (`internal/gatecli/green.go` `covered` and `recordPass`).
Either drop the guard and its row, or state that it is defensive.

F4. LOW, consistency, card 6 (`06-derive-entry-points.md`, the `DeriveDiff` Intent against its Tests paragraph). The git-read-failure branch is described two ways and nothing reaches it.
The Intent makes a failing git read a `Whole` fallback with a reason, as Decision 3 lists, and says "a cancelled `ctx` is the one error".
The Tests paragraph instead calls a failed `git diff` or `git show` "a propagated git error, which `refusal-spec.md` exempts as transient, so no row forces one".
A `Whole` fallback is a prescribed branch, not a propagated error, so the exemption does not cover it, and no row reaches it.
The card should pick one behaviour, and either add a reaching row or exempt it for a stated reason.

F5. LOW, design, card 6 (the `DeriveDiff` "base is empty" fallback). The fallback is not reachable from `gatecli`.
Card 8 runs a tier with no base whole without calling the derivation, and `diffBaseGroups` builds groups only from a resolved base or from `request.base`, which is non-empty under `--base` (`internal/gatecli/scope.go` `diffBaseGroups`).
Decision 3 lists this fallback, and `impactset.DeriveTargets` has the same branch, so treat it as a defensive branch the record requires.
Its test row reaches it only by calling `DeriveDiff` directly.

Tool calls: none; the inherited context held every file this lens needed.
