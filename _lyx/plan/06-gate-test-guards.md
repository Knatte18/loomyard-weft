# Card 6 — gate-test-guards

**Create:**
- `internal/gatecli#guardScanRefusal`

**Edit:**
- `internal/gatecli#newCommand`
- `internal/gatecli#testRequest`
- `internal/gatecli#runTest`
- `internal/gatecli/doc.go`
- `docs/overview.md`
- `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub`
- `internal/gatecli#TestGateTest_Scenario`

**Uses:**
- `internal/impactset#ModuleGuardTests`
- `internal/impactset#GuardScanError`
- `internal/impactset#GuardMisplaced`
- `internal/impactset#GuardNotIntegration`
- `internal/impactset#GuardParse`
- `internal/gatecli#goTestArgs`
- `internal/gatecli#startGate`
- `internal/gatecli#runGate`
- `internal/gatecli#runningGate.records`

**Intent:** Add D5's `--guards` flag to `lyx gate test`: after the named packages the verb runs every top-level test marked `//lyx:guard` in the module at the `-C` directory, by name in its own package, inside the same slot.

`newCommand` registers `--guards` (bool) and its help line, and the `test` help's examples gain `lyx gate test --tags integration --guards ./internal/foo`; `testRequest` gains `guards bool`.
In `runTest`, with `--guards` set:
- the empty-package refusal does not apply, so `lyx gate test --tags integration --guards` runs only the guard tests;
- a tag list without `integration` is refused first, before the package-directory check and hub resolution, naming `--guards`, that every guard compiles under `integration`, and the way forward to add `integration` to `--tags`; the input that reaches it is `lyx gate test --guards ./pkg` or `--tags tmux --guards`, and `TestGateTest_RefusesBeforeResolvingTheHub` gains both rows, the first listing `pkg` among its `dirs` so only the tag check can refuse it;
- after the package-directory check and before hub resolution and any slot, `impactset.ModuleGuardTests` scans the `-C` directory, and a scan error refuses the verb with `guardScanRefusal`'s text.
`guardScanRefusal` names the file for a `GuardScanError` and its line when `Line` is non-zero, with the way forward its `Kind` gives: move the marker directly above its `func Test…` line for `GuardMisplaced`; compile the marked test's file under the `integration` tag for `GuardNotIntegration`; make the file readable or fix it until it parses for `GuardParse`.
Any other scan error is named with the module directory and the way forward to make the directory readable or fix the module until `go list ./...` succeeds there.
Inside the held slot, after the prebuild, the verb runs `go test` over the named packages when there are any, then one `go test` per guard package in directory order, its arguments `goTestArgs` gives for that package's `./<dir>` and the flags after `--`, followed by `-run '^(<names>)$'`, so the guard's own pattern wins over a `-run` among those flags.
Each run is a tied child under the CPU watchdog as the package run is, and the verb stops at the first non-zero exit and returns that code, or 0 when every run passes.
With no packages named and no marked test in the module, the verb spawns no `go test`, logs that no guard test was found, and exits 0.
Without `--guards` the verb behaves as before.

`docs/overview.md`'s gate module line gives the synopsis with `[--guards]` and says the package list may be empty with it.

The flag widens what one verb call runs, never what it skips: it adds runs of tests the repository already holds, refuses a tag set the guards cannot compile under, and runs only inside the slot the verb already holds or inherits.
Bound: an unmarked scanning test is not run, as for the round gate.

`doc.go`'s verb section describes the flag, the scan, its refusals and the run order.

`TestGateTest_Scenario` gains rows over a nested module written under the prime, holding a package `pkg`, a package `g` with one marked test `TestGuard` and a package `h` with one marked test `TestGuardH`:
- `--tags integration --guards -C <module>` with no packages records two stand-in test invocations, `./g` with `-run '^(TestGuard)$'` then `./h` with `-run '^(TestGuardH)$'`, in directory order;
- with `./pkg` named it records the package invocation first and the two guard invocations after it;
- with `./pkg` named and `-- -run X` after the packages, each guard invocation carries `-run X` before its own `-run '^(…)$'`, so the guard's pattern is the last and wins;
- with `./pkg` named and the stand-in exiting 3, the verb exits 3 and records only the package invocation;
- over a second nested module with no marked test, `--tags integration --guards` with no packages exits 0 and records no invocation;
- a misplaced marker, a marked test in a `//go:build tmux` file, a test file made unreadable and a package directory made unreadable each exit 1 with an error naming, respectively, the file and line with the move-the-marker way forward, the file and line with the compile-under-`integration` way forward, the file with the make-readable-or-fix way forward, and the directory with the `go list ./...` way forward; the stand-in records no test invocation and the slot is free, the unreadable rows skipping as root.

**ImpactSummary:** the gate verb gains one opt-in flag whose scan and extra runs happen only when it is set; every existing invocation, the derived command included, is unchanged.
