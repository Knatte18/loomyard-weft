---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:124 (Decision 7, Linux bullet)
    summary: The warm-node claim rests on an unresearched fact; the tier child's stdout is a Go pipe with no WaitDelay, so a node or compiler server that inherits it holds Wait until it idles out
    fix: Research whether MSBuild worker nodes and the Roslyn compiler server (VBCSCompiler) redirect or close their inherited stdio when launched by `dotnet build` on Linux and Windows, and record the answer in Decision 7 with its source and SDK version (for example MSBuild's NodeLauncher stream redirection and the issue that introduced it). If they do not detach, add to Decision 6 that the build and tier runs set a bounded wait for lingering output holders on the tied child, stating that bound; either way add an acceptance criterion that a build which leaves a reused node running returns when its own process exits.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:62-63 (Decision 3) and :102 (Decision 6), :190 (acceptance criterion)
    summary: An empty test scope from a non-empty seed runs nothing when no opted-in build runs, so a card gate stays green over a compile break (e.g. a card that renames a type its tests still name), contradicting Decision 3's rationale and the unconditional acceptance criterion
    fix: In Decision 3, replace the empty-test-scope sentence with a rule that, under the `dotnet` scoper, a non-empty seed that selects no test class runs the build step when a listed tier opts in with `shared_build`, and otherwise runs each listed tier whole (the reason printed on stderr like the other fallbacks). Restate the acceptance criterion at line 190 with both branches.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:102 (Decision 6, first bullet) and :158 (PATTERN-gate-slot Constraint)
    summary: The build is said to run inside the slot hold of an empty-scope opted-in tier, but an empty-scope tier takes no slot today, and nothing tells a seed-empty scope from an inert-only one
    fix: Add to Decision 6's first bullet that an opted-in tier whose `dotnet` scope is the empty test scope from a non-empty seed acquires its tier's slots to run the build, and runs no test command inside them. State in Decision 2's result bullet that the derivation reports a non-empty seed apart from an inert-only or empty diff, so the call can tell the two empty scopes apart.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:107 (Decision 6, failed-build bullet)
    summary: A failed build is read by `gateslot.FailedTier` and Merge-In and Finalize's landing failure record as a test-tier failure with no failing tests, because the build prints no line of its own that the parser recognises
    fix: Add a bullet to Decision 6 that the build prints its own summary line, distinct from the `tier<N>` summary line, and that the opted-in tiers it blocks print no failed tier summary line. State that `FailedTier` and the landing failure record name the build as the failing run (a build kind or tier value the record states) and leave failing tests empty. Add an acceptance criterion that a failed build at Merge-In writes a landing failure record naming the build.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:42-46 (Decision 2, test-class rule and Bound)
    summary: The test-class rule under-matches, since a class that inherits its test methods from a base fixture carries no attribute, so it is never selected, and the base class's filter term cannot match inherited tests named after the derived class; MSTest DataTestMethod is also missing
    fix: Extend Decision 2's test-class rule so a non-abstract class whose declared base class (transitively, within the scanned sources) is a test class is itself a test class with its own filter term, and a selected abstract base class selects every class deriving from it. Add `DataTestMethod` to the method attribute list. Add an acceptance criterion for a derived fixture whose tests live in an abstract base.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:46-48 (Decision 2 Bound) and :63 (Decision 3 Rationale)
    summary: The Bound's list of misses leaves out dispatch through a base type or interface (a test that names only `Shape` or `IFoo` while the change is in `Circle` or a DI-resolved implementation), and Decision 3's claim that every unseen reach runs whole is false
    fix: Add dispatch through a base class or interface, including DI-resolved implementations, to Decision 2's Bound list of misses that the whole gates catch. Reword Decision 3's rationale to say that every case the derivation can detect runs whole, and that the misses listed in Decision 2's Bound are left to the whole gates of Decision 10.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md:133, :139 (Decision 8)
    summary: The claim that a carry false tier always runs whole is broken by paths outside the green record; verifytree.Verify skips a whole gate call on an unchanged tree, darn's tree-scoped sites call with explicit targets so the tier runs narrowed or not at all, and dropCovered drops a covered tier
    fix: >-
      In Decision 8, narrow the claim to the scope forms and the green record inside one `lyx gate test` call, and add a Bound naming what still skips or narrows the tier. That covers the verify site's verified-tree record on an unchanged tree, explicit targets including darn's tree-scoped calls, and a listed tier that covers it. Either have explicit targets also run a `carry: false` tier whole or state that the hub keeps such tiers out of the darn gate keys. Make the note `regression-baseline-verify` sentence match.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:173 (Auto-mode assumptions, second bullet)
    summary: The narrowed tier0 design rests on VSTest exiting 0 for a gated project built as a non-test project, which is assumed without evidence
    fix: Research what `dotnet test <project> --no-build --filter <expr>` does under VSTest for a project whose IsTestProject is false (exit code, message), citing the .NET SDK's test targets at the SDK version Models uses. Move the answer with its source into Decision 4 or the Constraints and delete the assumption. If the run fails, add a Decision that the derivation omits a test project the tier cannot run, or that the hub passes the gate properties in the tier's command.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:178 (Open risks, first bullet)
    summary: The MTP risk rests on a false premise, since a composed filter such as (Category!=Slow)&(FooTests) can match zero tests in a project the tier does run, and it leaves a known MTP failure undecided
    fix: Research Microsoft.Testing.Platform's exit code for a run that executes zero tests and the option that accepts it (exit code 8 and `--ignore-exit-code 8` or `--minimum-expected-tests`, per Microsoft Learn's MTP exit-code page), and record it with its source. Replace the risk with a Decision that `docs/dotnet-hub.md` tells an MTP hub to put that option in its narrowed tiers' commands. Correct the premise to include a tier filter that excludes every test of a selected class.
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:86 (Decision 5, first bullet) and :191 (acceptance criterion)
    summary: Decision 5 says an inert-only scope runs no listed tier's command, while Decision 8 says a carry false tier runs whole even on an inert-only diff, and neither carves out the other
    fix: >-
      Add to Decision 5's first bullet that a `carry: false` tier still runs whole under an inert-only empty scope (Decision 8), and add the same exception to the acceptance criterion at line 191.
  - id: F11
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md:201 (acceptance criterion, tier1 dotnet test)
    summary: The tier1 test runs `dotnet test` over a fixture solution, which needs NuGet test packages (NUnit, its adapter, Microsoft.NET.Test.Sdk) restored from the network, yet its skip conditions cover only a missing dotnet or SDK, so an offline or cache-cold machine fails the test
    fix: Restate the acceptance criterion so the test restores with no network source (for example `--source` pointing at the local NuGet global-packages folder) and skips, naming the reason, when that restore finds no test packages. This matches the existing precedent `TestCSharpFixtures_CompileWithDotnet`, which builds with no package reference.
  - id: F12
    severity: LOW
    class: decision
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md:75 (Decision 4)
    summary: The gate's shell-safe argument length names no existing bound, so the plan writer must invent the over-long filter limit
    fix: Replace the phrase with a stated limit in bytes of the rendered command, chosen so that it stays under cmd's 8191-character command-line limit on Windows. Say whether it reuses `greenCommandBudget` (6000 bytes) or is a new constant.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:134 (Decision 8) and :202 (acceptance criterion)
    summary: Adding carry, filter and shared_build to the fixed fingerprint struct changes every existing tier fingerprint, so the first run after landing voids every hub's green record, against the unchanged-behaviour criterion
    fix: Add to Decision 8 that the new keys join the fingerprint only when set to a non-default value, as Decision 5 does for a non-empty `inert_paths`, so an existing tier's fingerprint is unchanged. Alternatively, state the one-time loss as accepted and narrow the acceptance criterion at line 202 to say so.
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md:61-62 (Decision 3) and :191 (acceptance criterion)
    summary: What the dotnet scoper does on an empty diff is never stated, and the criterion's as-today wording has no today for a new scoper
    fix: Add to Decision 3 that an empty diff under the `dotnet` scoper yields the empty scope with no build, as an empty diff does under `go`, and restate the criterion's empty-diff clause for `dotnet` explicitly.
  - id: F15
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md:67-71 (Decision 4)
    summary: Decision 4 allows a filter placeholder under any scoper but defines its rendering only for whole, explicit, darn tree-scope and narrowed dotnet scopes, leaving the narrowed or card go scope and the guard step undefined
    fix: Add to Decision 4's rendering list that every scope other than a narrowed `dotnet` scope, including a narrowed or `--card` `go` scope and the guard step, renders the tier's `filter` alone, and nothing when the tier has none.
---

The record is well-grounded in the gate code and its review history shows the first lens pass was absorbed.
The findings below are design gaps and unresearched facts the plan writer would otherwise hit, none a PATTERN violation, so the verdict is APPROVED with every finding to fix.

### [MEDIUM] Warm build processes rest on an unresearched stdio fact

**Location:** `_lyx/discussion/decision-record.md:124`
**Issue:** `runInvocations` sets `cmd.Stdout = counter`, a non-file writer, so Go reads the child's output through a pipe, and neither `runTiedChild` nor `proc.StartTied` sets a `WaitDelay` (`internal/gatecli/tierrun.go`, `internal/proc/tied.go`).
If a reused MSBuild node or the compiler server inherits that pipe, `Wait` blocks until it idles out, turning Decision 7's warmth into a gate hang that ends on `timeout_min`.
The record names no evidence for how the SDK launches those processes; per the rubric that is a deferred research question.

### [MEDIUM] An empty test scope hides a compile break without an opted-in build

**Location:** `_lyx/discussion/decision-record.md:62-63`, `:102`, `:190`
**Issue:** Decision 3 promises the build runs for a non-empty seed that selects no class, but Decision 6 runs it only inside an opted-in tier.
With no `build`, or no opted-in listed tier, the call runs nothing.
A card that renames `Foo` to `Bar` seeds only `Bar` under `--card`, which reads no base version, so tests that still name `Foo` are never compiled and the card gate is green.

### [MEDIUM] The empty-scope build lands on a path that holds no slot

**Location:** `_lyx/discussion/decision-record.md:102`, `:158`
**Issue:** `runListed` hands an empty run to `runWithSlots(run, true, …)`, which skips the acquire (`internal/gatecli/run.go`).
Decision 6 never says that path now acquires slots, so as written the build runs unslotted, against the `PATTERN-gate-slot` Constraint.
The scope also carries no mark that tells a seed-empty scope from an inert-only one, though Decision 6 treats them differently.

### [MEDIUM] Landing sites misread a build failure as a test failure

**Location:** `_lyx/discussion/decision-record.md:107`
**Issue:** `landingshed.verifyGate.recordFailure` takes the tier from `gateslot.FailedTier`, which matches only `tier<N>` summary lines, or else uses the key's highest tier (`internal/landingshed/verifygate.go`, `internal/gateslot/runlog.go`).
After a failed build, the blocked opted-in tiers' failed lines, or the fallback, name a test tier with no failing tests, so the Merge-In and Finalize fixers are pointed at tests for a compile break.

### [MEDIUM] Inherited test methods are never selected

**Location:** `_lyx/discussion/decision-record.md:42-44`
**Issue:** `class Pipe1DTests : SolverTestBase {}` with `[Test]` methods only in the abstract base is no test class under Decision 2.
The base's filter term matches no test, because NUnit names inherited tests after the derived class.
This breaks the "never under-matching" guarantee and falls under no fallback.

### [MEDIUM] Decision 2's Bound omits polymorphic dispatch

**Location:** `_lyx/discussion/decision-record.md:46-48`, `:63`
**Issue:** The closure grows only toward files that name a seeded type.
A test that holds `Shape` or `IFoo` and receives a changed `Circle` or a DI-resolved implementation is never selected.
That is ordinary C#, not one of the four exotic forms the Bound lists, and Decision 3's rationale claims more than the derivation can see.

### [MEDIUM] `carry: false` is skipped or narrowed outside the green record

**Location:** `_lyx/discussion/decision-record.md:133`, `:139`
**Issue:** `verifytree.Verify` skips a whole call when the same command passed on HEAD's tree (`pattern/PATTERN-gate-slot.md`, Verified tree).
Darn's `darn-write`, `code-burler-fix` and darn Finalize calls pass explicit targets, which Decision 8 exempts, so a compare tier there runs narrowed or not at all.
`dropCovered` drops a listed tier that another listed tier covers.
Decision 8's "always runs" and its stated reason for note `regression-baseline-verify` hold only inside one call's green record.

### [MEDIUM] VSTest behaviour on a gated project is assumed

**Location:** `_lyx/discussion/decision-record.md:173`
**Issue:** A narrowed tier0 that selects a `NORCE.Models.SlowTests` class runs that project with `--no-build` and without `-p:RunSlowTests=true`.
Whether VSTest exits 0 there is left as an auto-mode assumption with no source, though the narrowed tier0 design rests on it.

### [MEDIUM] The MTP risk's premise is false

**Location:** `_lyx/discussion/decision-record.md:178`
**Issue:** With tier filter `Category!=Slow` and a selected class whose tests are all `Slow`, the composed expression matches zero tests in a project the tier does run.
The risk claims this cannot happen, and it leaves a known MTP failure with no decision.

### [MEDIUM] Decision 5 and Decision 8 contradict on an inert-only diff

**Location:** `_lyx/discussion/decision-record.md:86`, `:191`
**Issue:** Decision 5 and its criterion say an inert-only scope runs no listed tier's command, and Decision 8 says a `carry: false` tier still runs whole there.
The plan writer gets two rules for the same call.

### [MEDIUM] The tier1 dotnet test depends on the network

**Location:** `_lyx/discussion/decision-record.md:201`
**Issue:** Running `dotnet test` needs NUnit, its adapter and `Microsoft.NET.Test.Sdk` restored.
The skip covers only a missing `dotnet` or SDK, so an offline or cold-cache machine fails a gate test on live external state.
The cited precedent `TestCSharpFixtures_CompileWithDotnet` avoids this by building a library with no package reference (`cmd/lyx/csharpfixtures_integration_test.go`).

### [LOW] The over-long filter limit is unnamed

**Location:** `_lyx/discussion/decision-record.md:75`
**Issue:** No "shell-safe argument length" exists in the gate.
The only budget is `greenCommandBudget`, which covers carried targets, and cmd bounds the whole command line on Windows.

### [LOW] New fingerprint fields void every existing green record

**Location:** `_lyx/discussion/decision-record.md:134`, `:202`
**Issue:** `tierFingerprint` marshals a fixed struct (`internal/gatecli/green.go`), so new fields change every hash even when unset, unlike Decision 5's conditional `inert_paths` join.

### [LOW] Empty diff under `dotnet` is undefined

**Location:** `_lyx/discussion/decision-record.md:61-62`, `:191`
**Issue:** Decision 3 has no empty-diff case, Decision 5 excludes empty diffs, and "as today" has no meaning for a new scoper.

### [LOW] `{filter}` rendering is undefined for the `go` scoper's narrowed scopes and the guard step

**Location:** `_lyx/discussion/decision-record.md:67-71`
**Issue:** The placeholder is allowed under any scoper, but its rendering list stops at the four scopes it names.

## Focus directives

- Decisions 5 and 8 bounds: covered by F7, F10 and F13; the inert carry's own bounds hold against `greenRecord.with`, `covered` and `newestWholeAncestor`.
- Decision 6 against Decisions 3 and 5: covered by F2, F3 and F14; the inert-only no-build rule agrees across the three.
- Auto-mode assumptions and Open risks: covered by F1, F8 and F9.
- Constraints against Decision 10: they hold together, since Merge-In and Finalize run separate gate keys, so `verifytree` does not skip Finalize, and the green record carries per tier and tree.
  The last Constraint's note stays where it is: it changes only a background line of `PATTERN-gate-slot`, not an entry's overview text, so it is no `## PATTERN changes` bullet under the discussion spec.
- Leanness: not a defect.
  The Scope's Documentation bullet lists what `docs/dotnet-hub.md` must hold, and Decision 9's rule texts are the lens content that page carries.
  The plan writer reads neither file but the decision record, so moving either would lose a requirement.
