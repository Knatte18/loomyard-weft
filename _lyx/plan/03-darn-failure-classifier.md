# Card 3 — darn-failure-classifier

**Create:**
- `plan:internal/loomcli#impactedSet` -> `type impactedSet struct`
- `plan:internal/loomcli#importPathSet` -> `func importPathSet(worktree string, targets impactset.Targets) (impactedSet, error)`
- `plan:internal/loomcli#ownedFailure` -> `func ownedFailure(failures []websterengine.VerifyFailure, tests []verifytree.FailedTest, set impactedSet) (bool, []string)`
- `plan:internal/loomcli#notOwnedRoute` -> `func notOwnedRoute(outside []string, set impactedSet) landingshed.FailureRoute`
- `plan:internal/loomcli#darnFailureRoute` -> `func darnFailureRoute(ctx context.Context, failure verifytree.PublishFailure, worktree string, mergeBase func() (string, error), derive func(ctx context.Context, worktree, base string) (impactset.Targets, error)) landingshed.FailureRoute`
- `plan:internal/loomcli#TestDarnFailureRoute` -> `func TestDarnFailureRoute(t *testing.T)`

**Edit:**
- `internal/loomcli#verifySource`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/testkit/envkit#nilLegal`

**Uses:**
- `internal/loomcli#pairMergeBase`
- `internal/impactset#DeriveTargets`
- `internal/websterengine#ParseVerifyFailures`

**Intent:** D1 and D2 in the CLI wiring: a darn verify failure is classified as owned or not owned by the task, and a not-owned one is routed to Merge-In with a reason that says it lies outside the task's impacted set.
The code goes in a new file `internal/loomcli/darnclassify.go`, whose header names what it holds.

`impactedSet` holds three sets keyed by import path: `packages`, the impacted packages; `guards`, each guard package's `//lyx:guard` test names; and `binary`, the packages whose tests import `internal/testkit/lyxbin`.
`importPathSet` maps `targets` onto import paths: for each `ModuleTargets` it reads the module path from `<worktree>/<Dir>/go.mod` with `modfile.ModulePath` (`golang.org/x/mod/modfile`, already a module dependency), and spells `.` as the module path and `./x/y` as `<module path>/x/y`, so a nested module's packages carry its own module path.
`Guards` and `BinaryTests` are directories relative to the worktree in the root module, so they map onto the root `go.mod`'s module path; a missing or unreadable `go.mod` is the returned error.
The nested-module mapping and the `go.mod` read errors are defensive: `impactset.DeriveTargets` today emits only the root module and falls back to Whole for a nested module's file or a missing root `go.mod`, so only the fake `derive` of the test reaches them, and they keep the classifier correct if the derivation ever lists a nested module.

`ownedFailure` returns true, or false with the sorted distinct import paths of the failing packages, over the identities `websterengine.ParseVerifyFailures` reads from the log and the record's tests.
It returns true when any of these holds: `tests` is empty; an identity is opaque; a test identity's package is in `packages`; a test identity's top-level test name (the part before the first `/` of the ID after `<package>.`) is listed under its package in `guards`; a package identity's package, a build failure or a crashed test binary, is in `packages`.

`notOwnedRoute` returns target `Merge-In` (`loomshed.NameMergeIn`), the lead ` outside the task's impacted set, in <packages joined by ", ">,`, and this way forward:
`if the failure also fails on the parent branch, fix it there, then run "lyx loom goto --to Merge-In", then "lyx loom resume", in the task worktree, which merges a moved parent and re-runs the failing gate; if the task did cause it, run "lyx loom goto --to Darn-Write", then "lyx loom resume", in the task worktree, where a moved parent is merged in first`.
When any outside package is in `binary`, the way forward ends with `; <those packages> drive the lyx binary in their tests and may reach the task's change, so when the test does not fail on the parent branch, the Darn-Write goto is the way forward`.
Every step is a `lyx loom` verb the agents' settings allow, per `PATTERN-no-denied-recovery`.

`darnFailureRoute` returns the owned route, target `Darn-Write` (`loomshed.NameDarnWrite`) and the darn source's existing way-forward text, unless every check below lets the failure through as not owned:
it reads the log at `failure.LogPath` (an unreadable log is owned, logged at Warn), reads the merge base through `mergeBase` and derives `derive(ctx, worktree, base)` (either error is owned, logged at Warn), treats a Whole answer as owned for any fallback reason, maps the answer through `importPathSet` (its error is owned, logged at Warn), then asks `ownedFailure`; not owned returns `notOwnedRoute`.
The darn source's way-forward string moves into a package constant both `darnVerifySource` and `darnFailureRoute` read, so the owned route and the nil-classifier fallback keep one text.

`verifySource` gains `classify func(ctx context.Context, failure verifytree.PublishFailure) landingshed.FailureRoute`, nil for loom's source.
`darnVerifySource` sets it to `darnFailureRoute` over `l.WorktreePath()`, `pairMergeBase(l)` and `impactset.DeriveTargets`, the derivation Darn-Write's gate runs; `planVerifySource` leaves it nil, so the loom recipe keeps target `Webster-Burler-Review` and its way forward.
`landingDeps` copies it into `Deps.ClassifyVerifyFailure`.
`TestLandingDeps_EveryFieldPopulated` skips `ClassifyVerifyFailure` beside `PublishSkip` and asserts it is nil for loom's source; `nilLegal` gains `"Landing.ClassifyVerifyFailure": true`, so `envkit.NilSeams` over a loom wiring stays clean.
`TestWire_DarnRecipe`'s darn subtest asserts `c.verifySource().classify` is set, and its loom subtest asserts it is nil.

`TestDarnFailureRoute` is table-driven over a temp worktree holding a root `go.mod` (`module example.com/m`) and a nested `tools/go.mod` (`module example.com/tools`), a log file per row written with go test output lines, and a fake `derive` returning a told `impactset.Targets` with `Modules` `{".", ["./internal/a", "./internal/bin"]}` and `{"tools", ["./x"]}`, `Guards` `{"internal/g": ["TestGuard"]}` and `BinaryTests` `["internal/lyxdriver"]`.
Rows, each asserting the whole route.
Every owned row whose cause is not the failing package itself carries a record test and a log failure in `example.com/m/internal/z`, outside the set, so the row turns not owned if the branch it names were dropped:
- a failing test in `example.com/m/internal/a`, given by import path against the module-relative `./internal/a`, is owned;
- a failing test in `example.com/tools/x`, a nested module's package, is owned;
- a failing guard test `TestGuard/sub` in `example.com/m/internal/g`, outside the set, is owned;
- a failing test `TestOther` in `example.com/m/internal/g`, a guard package whose listed guard is `TestGuard`, is not owned, so the guard match is by name;
- a Whole answer is owned;
- a log holding `FAIL\texample.com/m/internal/a [build failed]` beside a failing test in `example.com/m/internal/z` is owned;
- a log yielding only the opaque identity, with the record still naming a test in `example.com/m/internal/z`, is owned, so the opaque branch is reached past the empty-tests check;
- a record with no failing tests is owned;
- a derive error is owned, and so is a merge-base error;
- a record whose `LogPath` names no file is owned;
- a missing `tools/go.mod` is owned;
- a log whose only failures, a package-level failure and a test failure, lie in `example.com/m/internal/z` and `example.com/m/internal/y` gives target `Merge-In`, the lead naming both, sorted, and the way forward naming `lyx loom goto --to Merge-In` before `lyx loom goto --to Darn-Write`, with no lyx-binary clause;
- a failing test outside the set in `example.com/m/internal/lyxdriver` gives target `Merge-In` and a way forward ending with the clause that it drives the lyx binary and names the Darn-Write goto.
The Warn rows assert nothing about the log; the route is the observable outcome.

Bound: the classifier changes only the record's target and the reason's way forward; every uncertain case counts as owned, so only a failure whose packages the import graph shows outside the diff's reach skips a Darn-Write trip, and that one reaches the parent after at most one Merge-In re-entry (D2).

**ImpactSummary:** `verifySource` gains one field read by `landingDeps` only; the drift guard and `nilLegal` absorb the nil loom value, and no other caller of the darn source changes.
