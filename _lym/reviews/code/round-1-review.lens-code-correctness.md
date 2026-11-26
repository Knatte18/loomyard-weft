```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced every function and branch the diff adds or changes.

## Cases run

- **`withTimeoutMin`** (`internal/gatecli/tierrun.go:245`):
  - **Correct.** A zero timeout cancels at once, as `context.WithTimeout(0)` did before.
  - **Correct.** A wall reading 21 h past start cancels with `errTimeoutMinPassed` at the first poll.
  - **Correct.** The wall clock running backwards falls back to the monotonic difference through `max`.
  - **Correct.** `cancel()` leaves the cause `context.Canceled`, so a tier that ended normally is never read as timed out.
  - **Correct.** When the parent ctx is cancelled, `Cause` is the parent's and the `ctx.Err() == nil` guard rejects it.
  - **Correct.** On a race between a non-zero exit and the bound passing, the run may read as timed out. The old `DeadlineExceeded` check had the same race, so this is not new.
  - **Correct.** The goroutine leaves on `bounded.Done()`, and the pending `after` channel is dropped.
- **Readers of `tierCtx` and `buildCtx`:**
  - **Correct.** `prebuildLym`, `tierTestTargets` and `exec.CommandContext` use only `Err()` or `Done()`, and no callee reads `Deadline()` or `DeadlineExceeded`.
- **`runBuild` and `runTierCommands` `timedOut`:**
  - **Correct.** A cause of `errTimeoutMinPassed` with a non-zero exit sets timed out; an exit of 0 never does.
- **`runTierCopies` prebuild** (`copies.go:66`):
  - **Not correct.** It still uses `context.WithTimeout`. See finding 2.
- **`runTiedChild` env:**
  - **Correct.** A nil `cmd.Env` becomes `os.Environ()` plus the marker.
  - **Correct.** The build always passes a non-nil env.
  - **Correct.** The shared `env` slice is cloned, so invocations never alias it.
- **`newChildMarker`:**
  - **Correct.** Concurrent copies get distinct numbers.
  - **Correct.** A nested gate's marker has its own pid.
  - **Correct.** The gate process never carries its own marker, so it never kills itself.
- **`killMarkedLeftovers`:**
  - **Correct.** On Windows, `ErrUnsupported` returns early.
  - **Correct.** An `ESRCH` on a process already gone is logged at Debug.
  - **Correct.** A process of another user, whose environ cannot be read, is skipped.
  - **Correct.** In the memory-cap scope path the holder has already been released, and killing it is harmless.
  - **Correct.** When the start failed, the scan runs and finds nothing.
  - **Not correct.** A process the leftover forks after the scan inherits the marker and survives the one pass. See finding 3.
- **After a timeout:**
  - **Correct.** The group kill fires and the escaped test host holds the pipe. `WaitDelay` returns after 10 s, and the marker kill then ends the host.
- **`vstestCounter.observeSummary`:**
  - **Correct.** The detailed-logger "Passed" and "Failed" blocks sum per outcome, and the `Total time:` line closes the block.
  - **Correct.** CRLF endings still match through `\s*$`.
  - **Correct.** Both invocation boundaries reset `inTotals`.
  - **Not correct.** A foreign line between `Total tests:` and its outcome lines ends the block, and those counts are lost. An indented `Total tests: N` in echoed test stdout adds N. See finding 1.
- **`observeFailure`:**
  - **Correct.** A `Total tests:` line closes an open failure block.
- **`takeCallerFilters`:**
  - **Correct.** It takes `--filter X`, `--filter=X` and `--filter:X`.
  - **Correct.** `--filter ""` and `--filter=` give `""`, which `filterArgument` drops.
  - **Not correct.** A trailing `--filter` with no value stays in `rest`. See finding 5.
- **`filterArgument` and `andFilters`:**
  - **Correct.** A single filter is left bare.
  - **Correct.** Tier filter, caller filter and terms render as `(tier)&amp;(caller)&amp;(t1|t2)`, and a caller's `A|B` keeps its parentheses.
  - **Correct.** Past the budget, the result is tier and caller without the terms.
  - **Correct.** An empty tier filter with a caller filter gives the caller filter alone.
- **`renderTierCommand`:**
  - **Correct.** A command without `{filter}` keeps the words in `{args}`.
  - **Correct.** A TRX run's prepended results args survive the extraction.
- **Planning refusal for a command's own `--filter`:**
  - **Correct.** It fires only without `{filter}` and with the `--filter` substring in the command.
  - **Correct.** A command holding both `{filter}` and a literal `--filter` is not refused and still gets two filters. That is a config conflict that existed before.
- **`holdsCallerFilter`:**
  - **Correct.** `-p:VSTestTestCaseFilter=` and `/p:` match case-insensitively.
  - **Not correct.** A bare trailing `--filter` counts as a filter. See finding 5.
- **`noTestNotice` and `filteredNoTestNotice`:**
  - **Correct.** A single invocation with other tiers declared, and one with none, give the right wording.
  - **Not correct.** In a multi-project narrowed run, the line is wrong for an empty project when another project of the tier matched. See finding 4.
- **`withOtherTiers` and `notRunTiers`:**
  - **Correct.** Covered tiers are excluded.
  - **Correct.** Guard and vet runs are not counted.
  - **Correct.** Both return paths set the field.
  - **Correct.** With `--tier 3` alone, tier0 is listed.
- **`planTierRuns` tier0 check:**
  - **Correct.** The message carries `gate.tiers.tier0`, as `TestPlanTierRuns` pins.
- **`Config.Tier` and `Config.Gate` messages:**
  - **Correct.** Callers wrap them as `gate test: %w`, and the text stays well formed.
- **`TestWithTimeoutMin`'s fake clock:**
  - **Correct.** A nil `after` channel blocks until cleanup cancels.

## Findings

1. **LOW, design.** `internal/gatecli/counters.go:224` (`observeSummary`).
   The multi-line summary closes at the first line that is not an outcome line.
   - An interleaved line between `Total tests: N` and its `Passed:`/`Failed:`/`Skipped:` lines drops that summary's outcome counts. Such a line can be stderr, or another project's output when `dotnet test` runs several projects in parallel.
   - Because the regex accepts leading whitespace, an indented `Total tests: N` that the detailed logger echoes from a test's standard output adds N to `discovered`.
   - Evidence: `counters.go:164`, `:168`, `:243-250`.
2. **LOW, design.** `internal/gatecli/copies.go:66`.
   A `--copies` run's shared prebuild is still bounded by `context.WithTimeout`, which runs on the monotonic clock, so it does not end across a host sleep.
   The changed `doc.go` says the tier's `timeout_min` bounds the prebuild and counts wall-clock time (`doc.go` hunk at `:316-319`).
3. **LOW, design.** `internal/gatecli/child.go:109` (`killMarkedLeftovers`).
   It scans once and kills once. A marked leftover that forks between the `/proc` scan and its SIGKILL leaves a marked child that outlives the run.
   The doc's Bound (`doc.go:582-586`) does not name this window.
   Evidence: `proc/envmarker_linux.go:17-40`.
4. **LOW, design.** `internal/gatecli/tierrun.go:657` and `:518-528`.
   The notice is built per invocation and says "no test of tier0 matched the filter" with "the test may belong to" another tier. In a narrowed dotnet run with several projects, a caller filter that matched tests in one project prints that line for each other project. The claim about the tier is then false.
5. **NIT, design.** `internal/gatecli/tiers.go:925-938`.
   A trailing `--filter` with no value is not taken out. It renders beside `{filter}` as a second `--filter`, and planning does not refuse it. `holdsCallerFilter` (`tiers.go:942`) still treats it as a caller filter.

Tool calls: none; the manifest's context covered every function traced.
