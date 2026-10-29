# Decision record — test-prune-audit-leads

## Goal

Act on the leads of the first recurring test-time audit: make `internal/reedengine`'s `tier2` tests run in a fraction of their time while asserting the same behavior, and prune redundant tests in `internal/shell`, `internal/boardengine/boardtest` and `internal/battenrecipe`.

## Scope

- In: `internal/reedengine`'s pane-binary seam in `internal/reedengine/panebin.go`, its three production callers and the tests that inject it.
- In: `t.Parallel` for every `internal/reedengine` `tier2` test that touches no process-global state.
- In: a redundancy read of the tests of `internal/shell`, `internal/boardengine/boardtest` and `internal/battenrecipe`, and the folds it finds.
- In: a correction note in the audit section of `docs/benchmarks/test-suite-timing.md`, where its counts for the three packages and its reedengine reading are wrong.
- Out: the other prune candidates the audit lists (`internal/shedrun` and the packages whose growth it judged matched).
- Out: `t.Parallel` for tests outside `internal/reedengine`, beyond tests a fold already rewrites.

## Decisions

### D1 — reedengine's tier2 time is a serial sum, not a waited-out duration

A verbose gate run of the package's `tier2` tests shows 32 top-level tests whose times sum to the package's wall time, about 58 s, the longest 11 s, with no test waiting out a fixed production duration.
None of them calls `t.Parallel`, so the speed-up the board entry asks for comes from running them in parallel, which `PATTERN-test-speed`'s "Tests run in parallel" already requires.
Rationale: there is no single wait to inject, and the audit's "2 tests, 40.00 s" is a reading of interleaved log lines.

### D2 — The executable-path seam becomes an engine field

The package-level `executablePath` variable becomes an `Engine` field that `New` sets to `os.Executable`, and the compose functions in `internal/reedengine/panebin.go` take the lookup as a parameter.
A test sets the field on its own engine, or passes the lookup to a compose function, instead of swapping the package variable.
Rationale: `PATTERN-test-speed`'s "Time is injectable" keeps a seam off package-level vars so tests keep `t.Parallel`; this seam is what holds most of the slow tests serial.
Bound: production behavior is unchanged, since every engine `New` builds resolves the path through `os.Executable` exactly as before.

### D3 — Tests on process-global state stay serial and say so

A `tier2` test that calls `t.Setenv` or captures the process logger through `logcapture` stays serial, with a comment naming that state, as `PATTERN-test-speed` asks.
Every other `tier2` test of the package, and every `tier0` test that was serial only because of the seam, calls `t.Parallel`.

### D4 — The three packages' growth is far smaller than the audit says

Counting `func Test` declarations, the packages grew since 2026-10-07 from 11 to 14 tests in `internal/shell`, from 3 to 3 in `internal/boardengine/boardtest` and from 8 to 9 in `internal/battenrecipe`.
The audit's 65, 68 and 57 are not these packages' tests: `internal/shell` holds 14 tests with 60 subtests, `boardtest` 3 tests with 7 subtests, and `battenrecipe` 9 tests with none.
`internal/shell` keeps one table per `Shell` method, each pinning a distinct method's output in both dialects, and `boardtest` keeps its three distinct contracts; both are left as they are.
Rationale: the board entry leaves a package whose tests each pin distinct behavior unchanged.

### D5 — battenrecipe folds the Run-Shed rename guard into the row-name test

`TestNameRunShed_ValueIsPinnedAgainstASymmetryRename` repeats the row-name loop of `TestNew_RowNamesMatchTheDurableIdentityConstants`; its one distinct assertion, that `NameRunShed` reads `Run-Shed`, moves into the row-name test, and the duplicate is deleted.

### D6 — The audit section gets a correction note

The audit section in `docs/benchmarks/test-suite-timing.md` gets a short correction under its prune-task candidates naming the real counts and the reedengine reading, so the next audit does not repeat the miscount.
Rationale: the section is a measurement report and its numbers stay, but a reader acting on its leads needs to know two of them are wrong.

## Constraints

- `PATTERN-test-speed`: every test keeps its tier tag; no test waits out a production duration; tests run in parallel unless they touch process-global state, which a comment names.
- `PATTERN-shell-guard-table`: `internal/shell` keeps importing only the standard library; no test change touches its code.
- `PATTERN-spawn-observability`: the compose functions keep logging an unresolvable executable through `logger.Warn`.

## Auto-mode assumptions

- Each `tier2` test builds its own tmux server on its own socket under a fresh temp directory, so parallel tests share no server.
- The audit's per-package numbers came from interleaved log lines; the counts above are read from the source and from a verbose gate run.

## Open risks

- Parallel `tier2` tests raise the load on the tmux servers they start, so a wait bounded at a few seconds could time out on a loaded host; the gate runs the package several times before landing to check.

## Acceptance criteria

- D1, D3: a gate run of `internal/reedengine`'s `tier2` tier passes with its package time well under the serial 58 s, and every test that does not call `t.Parallel` carries a comment naming the process-global state it touches.
- D2: no package-level variable holds the executable lookup; the pane launch line, the window command and the switch and follow commands still carry the injected path, and an unresolvable path still drops the prelude or the binding, each asserted by the existing tests.
- D4: `internal/shell` and `internal/boardengine/boardtest` tests are unchanged.
- D5: `internal/battenrecipe` holds 8 top-level tests, and the row-name test asserts both the row order and `NameRunShed`'s durable value.
- D6: the audit section names the corrected counts and the reedengine reading.
