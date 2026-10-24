# Fixer report — discussion round 1

F12 is `decision`-class and went to the parent; this report leaves it out.
The parent's F12 decision, added to the record after Discussion, is applied in Decision 4 and its `{filter}` acceptance criterion: the over-long filter bound is `greenCommandBudget`.

## Changes

- F1 (fixed as proposed): researched the SDK's stdio handling.
  MSBuild's `NodeLauncher.cs` at `vs17.12` and Roslyn's `TryCreateServer` at `release/dev17.12` start nodes and the compiler server with stdio redirected on Unix, and with invalid standard handles and `bInheritHandles: false` on Windows.
  Decision 7 gains an Output bullet recording this with its sources, so no wait bound is added to Decision 6.
  It also gains a Bound on the escape hatches, and the `tier1` criterion now checks that the call returns.
- F2 (fixed as proposed): Decision 3 now says that a non-empty seed selecting no class runs the build when a listed tier opts in, and otherwise runs each listed tier whole with the reason on stderr; the criterion states both branches.
- F3 (fixed as proposed): Decision 6's first bullet says an empty-test-scope opted-in tier acquires its slots for the build and runs no test command, and Decision 2's result bullet says the derivation reports a non-empty seed apart from an empty or inert-only diff.
- F4 (fixed as proposed): Decision 6 gains a bullet for the build's own summary line.
  The blocked tiers print no failed `tier<N>` line.
  `gateslot` reads the build line beside `FailedTier`.
  The landing failure record carries `build: true`, with `tier` set to the first blocked opted-in tier and no failing tests.
  A new criterion covers Merge-In.
- F5 (fixed as proposed): Decision 2 adds `DataTestMethod`, makes a non-abstract class deriving transitively from a test class a test class with its own term, and makes a selected abstract base select its derived classes; new criterion for a derived fixture with tests in an abstract base.
- F6 (fixed as proposed): Decision 2's Bound lists dispatch through a base class or interface, DI-resolved implementations included; Decision 3's rationale now claims only detectable cases and leaves the Bound's misses to Decision 10.
- F7 (fixed as proposed, taking the "state that the hub keeps such tiers out" branch): Decision 8 narrows the claim to one `lyx gate test` call that lists the tier, with a Bound naming the verified-tree skip, explicit targets including darn's tree-scoped calls, and covering tiers.
  `docs/dotnet-hub.md` tells a hub to keep such tiers out of darn's gate keys and `covers`, and to run tiers that must run on an unchanged tree only through direct calls.
  The `regression-baseline-verify` sentence now matches, and so do the heading and the Scope bullet.
- F8 (fixed as proposed): Decision 4 records that the test platform's `VSTestTask` runs only when `IsTestProject` is `true` (microsoft/vstest `Microsoft.TestPlatform.targets` at `v17.12.0`, the .NET 9 SDK's), so a gated Models project tests nothing and succeeds; the auto-mode assumption is deleted.
- F9 (fixed as proposed): Decision 4 corrects the premise to include a tier filter excluding every test of a selected class, records VSTest's `TreatNoTestsAsError` default of false and Microsoft.Testing.Platform's exit code 8 with `--ignore-exit-code 8` / `TESTINGPLATFORM_EXITCODE_IGNORE`, and decides `docs/dotnet-hub.md` tells an MTP hub to set one of them on `{filter}` tiers; the open risk is deleted.
- F10 (fixed as proposed): Decision 5's first bullet, its criterion and the Scope bullet carve out a `carry: false` tier, which runs whole under an inert-only scope.
- F11 (fixed as proposed): the `tier1` criterion restores with `--source` naming the local global-packages folder and skips, naming the reason, when that restore finds no test packages; the test-isolation Constraint says the same.
- F13 (fixed as proposed): Decision 8 says `carry`, `filter` and `shared_build` join the fingerprint only when non-default, so existing fingerprints are unchanged.
- F14 (fixed as proposed): Decision 3 says an empty diff under `dotnet` yields the empty scope with no build; the inert criterion states the empty-diff clause per scoper, and Decision 6 and its criterion say "empty-diff" where they said "empty-seed" or "inert-empty".
- F15 (fixed as proposed): Decision 4's first rendering bullet covers every scope but a narrowed `dotnet` one, the `go` scoper's narrowed and `--card` scopes and the guard step included; the `{filter}` criterion matches.
- Support log: a Round 1 entry records the facts researched for F1, F8 and F9 with their sources.

## Self-check

- fix-consistency LOW, Decision 8 heading still says "always": fixed as proposed; heading now "runs whole wherever it is listed".
- fix-consistency LOW, `{filter}` criterion names only whole scope: fixed as proposed; criterion lists every scope Decision 4 now renders.
- fix-consistency LOW, build criterion omits the empty-diff case: fixed as proposed.
- fix-consistency LOW, Scope `inert_paths` bullet lacks the `carry: false` exception: fixed as proposed.
- fix-consistency LOW, Decision 2 result bullet promises a term for an abstract class: fixed as proposed; terms are per selected non-abstract class.
- fix-claims MEDIUM, escape-hatch Bound cited against `vs17.12`: fixed as proposed; the Bound now says it holds for main's launcher, newer than the .NET 9 SDK's, on Unix, and the support log matches.
- fix-claims MEDIUM, Decision 8 and Scope bullet overstate the bound given `dropCovered` and explicit targets within a call: fixed as proposed; both now except explicit targets and a covering listed tier, and the Bound opens "three paths still skip or narrow".
- fix-claims LOW, fix-way-forward MEDIUM and fix-proposal MEDIUM (merged), `tier1` criterion asserts a lingering node that cannot exist on Windows and needs process probing: fixed with my own change; the check is Linux-only and asserts the call passes under a `timeout_min` below the SDK's node idle timeout, with no probe for processes it did not start.
  I did not take fix-proposal's "keep only the call returns once its own processes exit" option, because every passing test already shows that and it pins nothing.
- fix-way-forward LOW, the note has no way to avoid the verified-tree skip: fixed as proposed; `docs/dotnet-hub.md` tells a hub to run a tier that must run on an unchanged tree only through a direct `lyx gate test` call.

## Disputed

None.

## Deferred

None.
