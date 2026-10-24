```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens. I traced the existing code paths that meet each state, setting or file the record's decisions introduce or change.

**Paths traced, per decision**

- **D1/D2/D3 (scoper `dotnet`)**
  - `computeScope` calls `cardPackages`, the Go-shaped `CardPackages`, before it branches on the scoper (`internal/gatecli/scope.go:122-128`): **survives**. The `none` scoper already passes this path with no use of its result, and the Notes name the branch point.
- **D4 (`{filter}`, tier `filter`)**
  - `renderTierCommand`/`quoteAll` → `verifyrun.QuoteArg`, which refuses a `%` value on Windows (`internal/verifyrun/run.go:65-72`): **survives**, since the filter grammar uses no `%`.
  - Darn's tree-scope call, which runs explicit targets (pattern/PATTERN-gate-slot.md, "Verified tree" bullets): **survives for D4**, which renders the tier filter alone there.
- **D5 (`inert_paths`, inert carry)**
  - `greenRecord.with` pruning keeps the 20 newest whole entries with empty args (`green.go:118-150`): **survives**, because the carry writes a whole entry at HEAD.
  - `covered` per tree (`green.go:158`): **survives**.
  - `newestWholeAncestor` as the computed base in `diffBaseGroups` (`scope.go:241-246`): **survives**. A changed list voids the ancestors, and the base falls back to the merge base.
  - `subtract` under a non-`go` scoper (`green.go:270-275`): **survives**. A whole entry drops every narrowed invocation in the tier's dir.
  - `reportCarried` (`run.go:392`), which writes a `Carried` record and takes no slot: **survives**.
- **D6 (build step)**
  - `runListed` → `runWithSlots(run, true, …)` for an empty scope, which takes no slot (`run.go:353-358`, `run.go:400-404`), and `runInvocations`, which returns at once with no invocations (`tierrun.go:117-119`): **breaks**, see F1.
  - `report` → `TierSummaryLine` → landing `recordFailure` → `gateslot.FailedTier`, else `slices.Max(tiers)` (`internal/landingshed/verifygate.go:96-105`, `runlog.go:190-203`): **breaks**, see F2.
  - `failingTestsOf` → `websterengine.ParseVerifyFailures` (`internal/loomcli/landingdeps.go:551`): reads a build failure as no failing test. This is part of F2.
  - `verifytree` → `FindTierTimeout` (`internal/verifytree/verify.go:477`): **survives**, since D6 states a build timeout line that `verifytree` reads.
  - `recordFailure` skips a `TimedOut` result (`verifygate.go:63`): **survives**, the same as a tier timeout.
  - `WriteRunRecord`, whose name stem is `tier%d` from the integer `RunRecord.Tier`, and `report.go`, which groups by `TierList`/`ScopeForm`: **survives**, because D6 names the build record and how it is reported.
  - `runTier`'s timeout, counted from the call and including the prebuild (`tierrun.go:51,67`): **survives**, because D6 restates where the tier's clock starts.
- **D7 (warm processes)**
  - `runInvocations` sets `cmd.Stdout = counter`, which is a pipe, with no `WaitDelay` (`tierrun.go:141-143`). `Tied.Wait` takes `cmd.Wait`'s result before `releasePlatform` closes the Windows job (`internal/proc/tied.go:65-70`): **at risk**, see F3.
- **D8 (`carry: false`)**
  - `verifytree.Verify` skips a whole call whose same command has a pass naming HEAD's tree (pattern/PATTERN-gate-slot.md, "Verified tree"): **breaks** the "always runs" claim, see F4.
  - Darn's tree-scoped `darn-write`, `code-burler-fix` and darn Finalize calls run explicit targets, and an empty such call runs nothing (pattern/PATTERN-gate-slot.md): **breaks** "a compare must never be narrowed", see F5.
  - `dropCovered` drops a listed tier that a higher listed tier covers (`tiers.go:254`): **holds**. The tier that covers it runs the command.
- **D8 and the Constraints (fingerprint)**
  - `tierFingerprint` hashes a fixed struct through `json.Marshal` (`green.go:64-82`): **breaks** the "unchanged" acceptance criterion once, see F6.
  - The green-record Constraint against `recordPass` (`green.go:345-353`): **survives**. A tier without `{targets}` is still recorded whole, which is right, because D4 runs it whole with its tier filter only.
- **D10 (gate keys)**
  - Merge-In and Finalize run separate gate keys and so separate commands (`internal/landingshed/deps.go:199-202`). The `verifytree` record does not skip Finalize, and the green record carries per tier and tree: **survives**.

**Findings**

- **F1 — MEDIUM, consistency, D6 first bullet.**
  D6 runs the build "inside the slot hold of the first opted-in tier … whose `dotnet` scope is an empty test scope from a non-empty seed".
  Today an empty-scope tier takes no slot, and its run returns before any command runs.
  D6 never states that this path now acquires the tier's slots and runs the build.
  As written, the empty-seed compile check it promises (D3, and the AC "runs the build step and no test command") lands on a path that holds no slot.
  Evidence: `internal/gatecli/run.go:353-358`, `run.go:400-404`, `internal/gatecli/tierrun.go:117-119`.
- **F2 — MEDIUM, design, D6 failed-build bullet.**
  After a failed build, each remaining opted-in tier "fails without running … with a stderr line and run record".
  The landing failure record takes `failure.Tier` from the first non-zero `tierN` summary line, else the highest tier of the key.
  Its failing tests come from the Go test-failure parser.
  A build failure is therefore recorded as a failure of the first opted-in test tier with no failing tests, and traced to cards as a test failure.
  D6 says nothing on how Merge-In and Finalize record a build failure, such as the record's kind or tier.
  Evidence: `internal/landingshed/verifygate.go:96-105`, `internal/gateslot/runlog.go:190-203`, `internal/loomcli/landingdeps.go:551-563`.
- **F3 — MEDIUM, design, D7.**
  D7 says idle MSBuild nodes outlive the tied child on Linux and end with the job on Windows.
  The tier's child writes to a Go pipe, `cmd.Stdout = counter`, with no `WaitDelay`.
  `Tied.Wait` returns only after `cmd.Wait` has seen EOF, and on Windows it closes the job only after that.
  If a reused node or the compiler server inherits the stdout handle, each tier or build run blocks until that process idles out, up to its timeout.
  On Windows the job never closes first, so the run blocks until the tier's timeout.
  The record cites no evidence that MSBuild and the compiler server detach from the parent's stdio, though D7's design rests on that fact.
  Evidence: `internal/gatecli/tierrun.go:141-143`, `internal/proc/tied.go:65-70`.
- **F4 — LOW, decision, D8.**
  "Neither a computed scope nor an inert-only or no-class diff can leave it unrun" holds only inside `lyx gate test`.
  At a verify site, `verifytree.Verify` skips the whole call when the same command already passed on HEAD's tree, and with it the `carry: false` tier.
  D8 states no bound for that skip, though it gives note `regression-baseline-verify`'s capture as a reason.
  Evidence: pattern/PATTERN-gate-slot.md, "Verified tree": "a skip requires an entry of the same command naming HEAD's tree".
- **F5 — MEDIUM, design, D8.**
  D8 exempts explicit targets, but darn's tree-scoped sites call `lyx gate test` with explicit targets.
  These are `darn-write`, `code-burler-fix` in a darn run, and darn Finalize.
  A `carry: false` compare tier listed in those keys runs narrowed to the touched trees' targets, and runs nothing for an empty call.
  That contradicts D8's own reason that "a compare must never be narrowed".
  The decision states no bound and no handling for this path.
  Evidence: pattern/PATTERN-gate-slot.md, the bullets on Darn-Write's tree-scoped call and darn Finalize's empty call; `internal/gatecli/tiers.go:164-169`.
- **F6 — LOW, consistency, D8 "`carry` and `filter` join the tier fingerprint" against the AC "Existing gate behaviour … unchanged".**
  `tierFingerprint` hashes a fixed struct with `json.Marshal`, so adding fields changes every tier's hash even when the keys are unset.
  The first call after installing the binary carries nothing and finds no green ancestor in any hub, loomyard included.
  The record does not say whether that one-time loss is accepted or whether unset keys stay out of the hash.
  Evidence: `internal/gatecli/green.go:64-82`.

**Tool calls**

1. Read `RunRecord`, `WriteRunRecord` and the `report.go` grouping, to check how a build record would be named and counted. My context had only `TierSummaryLine` and `FailedTier`.
2. Grepped for readers of `RunRecord`, `FailedTier` and `FindTierTimeout`, to find every path that reads a gate failure. The manifest named only one reader.
3. Read `failingTestsOf` and darn's `tree_scope` docs, to trace the landing failing-test parser and darn's explicit-target calls. Both were one hop past the manifest.
4. Read `proc/tied.go` `Wait` and the Windows job close order, to check whether lingering build processes holding the pipe block `Wait`. The manifest named the file but I had not read it.
