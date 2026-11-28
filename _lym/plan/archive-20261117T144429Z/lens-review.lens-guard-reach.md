```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, skip and refusal the plan's cards add, I checked whether a real entry point can reach the state it guards, and whether the guarded site recurs unguarded.

**Findings**

- **BLOCKING — Card 7, `TestLaunchSpecLiterals_AreListed` `RealModule` row (Table scan): it fails on real code that has no catalog entry.** The scan reports every `seatengine.Table` composite literal whose enclosing function no `seatTables` `Builder` names. Nothing makes that unreachable state impossible, and it already fires in several ways:
  - It sees the empty `seatengine.Table{}` returned on an error path. These sit in functions the catalog never names, for example:
    - `internal/loomcli/wiring.go:996`, `:1034`
    - `internal/loomcli/driftwiring.go:171`
    - `internal/loomcli/darnwiring.go:294`
    - `internal/loomcli/projectrecord.go:262`
    - `internal/shedrecipe/entries_planwrite.go:94`, `entries_parts.go:61`, `entries_raddle.go:78`
    - `internal/driftengine/expect.go:241`, `verify.go:653`
  - Some literals sit in a function other than the builder the catalog names:
    - `internal/landingshed/describe.go:176` is in `describeSessionTable`, not `DescribeTable`.
    - `internal/landingshed/fixreview.go:150` is in `fixReviewTable`, not the new `FixReviewSeatTable`.
    - `internal/burlerengine/review.go:72` and `fix.go:82` are in `reviewTable` and `fixTable`, not the new `Profile.SeatTables`.
  - `internal/shedrecipe/entries_multillm.go:91` is in `multiLLMEntry`. The overview decision deliberately leaves it out of the catalog, but no scan skip is planned for it.
  - The card names no rule that skips an empty literal, no list of literal-holding functions per entry, and no MultiLLM skip.
- **BLOCKING — Card 7, Spec scan `RealModule` row: `seatSpec`'s literal is unlisted.**
  - The row scans Spec literals "against `producers`' functions and `seatTables`' builders".
  - `internal/seatengine/prompt.go:225` (`internal/seatengine#seatSpec`) sets `Prompt`.
  - Card 7 says `seatSpec` "takes no entry here". `seatTables`' builders are table-builder glyphs, never `seatSpec`, and `internal/seatengine` is not among the skipped places.
  - So the scan flags this literal on the real module.
- **MEDIUM — Card 4 (and card 3): the heddle file items are read off the passage reader, not the launch.**
  - `heddle-file/<slug>` items come from `Standing.FileEntries` whether or not `AGENTS.override.md` (or `CLAUDE.local.md`) was rendered and is read.
  - The dropped-delivery state does occur. Heddle never writes over an unmarked, repository-owned `AGENTS.override.md` (`heddleengine.Marker`, `internal/shuttleengine/codexengine/instructionfiles.go` `heddleRendered`). Codex then never reads the `codex-file` and `agents-file` entries, yet the manifest still lists them.
  - The parity fixture (card 6) only installs a reader, so it cannot reach a dropped file delivery, and the agents-file acceptance row cannot fail on it.
- **MEDIUM — Card 8, verb's `Seats(env)`: stand-in inputs land outside the temporary directory.**
  - Card 6's builders take stand-in paths under `env.Dir`, and some read files there (`PartsPlanTable` calls `readPartsFile`).
  - Card 8 resolves `Seats` before `Report` makes its temporary directory, and names no `env.Dir` for the verb.
  - So either the stand-ins are written outside the temporary directory, breaking D10's "writes nothing outside", or a builder errors.
  - `Seats` returns one error for all tables, so one failing builder refuses the verb for every seat.
- **LOW — Card 7: the "forks required" skip in `TestProducers_CodexManifestHoldsClaudes` is unreachable.**
  - `producers` build specs "with the literal's role, config role and skills" only, not `ForkSubagents` or `Requires`.
  - Webster `Run`'s spec (`internal/websterengine/runlevel.go:758`) sets both, but a producer entry never carries them, so the skip never fires.
  - Codex declares no forks support (`internal/shuttleengine/codexengine/capabilities.go` has no `Supports`), so card 6's seat skip is reachable; card 7's is not.
- **LOW — Card 5: `TestNewWithSeams_CodexTakesTheGivenSeams` cannot reach the probe on the fixture as named.**
  - The test asserts the probe is called under `auto`.
  - `newCodexLaunchFixture` writes `sandbox_mode: danger-full-access` (`internal/shuttleengines/codexlaunch_test.go`), under which `launchSandboxMode` never calls the probe.
  - The card lists no fixture change or override.
- **LOW — Card 2, `copyManifest` in `Runner.launch`: copies outlive failed launches.**
  - The copy is written before the ledger write and `AddStrand`. A later failure removes the run directory (`internal/shuttleengine/run.go:651-737`) but leaves a copy for a launch that never ran.
  - Each start retry (`run.go:607`) leaves another such copy.
  - The card bounds only that nothing removes copies; it does not say failed attempts keep one.

**Further tool calls**
1. I grepped `codexengine/capabilities.go` for `Supports` and every production `seatengine.Table{` literal with its enclosing function. My context had the builders but not every literal site, nor Codex's requirement answer.
2. I read the run-root scans in `activity.go`, `agentwait.go`, `sessionread.go` and `waitmark.go`, plus the launch call sites in `run.go`. This checked whether the `manifests` directory recurs unguarded in a scan other than the two that card 2 guards; my context lacked those loop bodies. They skip it silently, so there is no finding there.
