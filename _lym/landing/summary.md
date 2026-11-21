# Gate: TRX verdicts, runnable darn measuring calls, tier env overrides

This change fixes three gate faults reported from .NET hubs.

## A vstest tier's verdict comes from a results file the gate asks for

The `vstest` counter used to read only VSTest's console summary and result lines.
If the call's own words after `--` included a detailed console logger, those lines took a different shape: the counter counted zero, and the gate printed "ran no test" even though the run reported `Passed: 1`.
A crashed test host also left no cause in the gate log.

Now a `vstest` invocation whose command holds `{args}` gets `--results-directory <dir> --logger trx;LogFilePrefix=gate --blame` at the front of its args, over a fresh `lymtemp` directory (`internal/gatecli/trx.go`, `runInvocations` in `tierrun.go`).
When the command ends, the TRX files in that directory replace the counts and failed tests the console output gave for that invocation (`vstestCounter.replaceInvocation`), and the directory is removed.
- Outcomes map as follows:
  - Passed counts as passed.
  - Failed, Error, Timeout and Aborted count as failed.
  - Every other outcome counts as skipped.
- A failed test keeps the first lines of its error message and stack trace.
- When a run fails, each run-level message in the TRX files is printed to stderr, written to the tier log and listed among the failure report's causes.
  This covers a crashed test host, together with the test the blame collector saw running.
- An invocation that wrote no readable TRX file keeps the console counts.
  So does a command without `{args}`, since the gate cannot reach that command's arguments.

## The darn prompt names only a measuring call the hub can run

The cold-fixer Step 4 list of the darn writer's prompt hard-coded `lym gate test --gate darn-write <its package or test project> -- <args>`.
A hub whose gate config refuses the key, or whose tiers take no targets, refuses that call.

`gateslot.Config.MeasureCall` now renders the call from the hub's config:
- The call is `--gate <key>` when `Config.Gate` accepts the key, and falls back to `--tier 0` otherwise.
- It includes the target word only when a tier the call runs takes targets.

The darn wiring loads the gate config each time it builds the table and fills the stencil's new `measure_call` marker.
A hub with no `gate.yaml` gets the zero config, whose measuring call is the `--tier` form, so its darn table still builds.
`DarnInputs.MeasureCall` is a new required input.
`Config.Gate`'s refusal of an unknown key now lists the keys the config holds, so `lym gate test --gate <unknown>` names the declared keys.

## A tier's env values that differ from the session's are named

Before its commands run, a tier run with an invocation now prints and logs one line naming each variable of the tier's `env` that the calling session sets to a different value.
The line shows both values and gives the way forward of changing `tiers.<tier>.env` from the prime.
A variable the session leaves unset is not named.
The session environment is the gate process's own, held on the request as `sessionEnv`.
The gate integration tests drop the session's `GOFLAGS` from the gate they start, so a developer's own value adds no env line to their expected output.

## Design decisions

- The results-file arguments use the `dotnet test` spelling and go only where the gate controls the args (`{args}`).
  Adding a config key to opt in would add more config for what every `dotnet test` tier wants.
- `--blame` writes a sequence file instead of a crash dump, so a crash names its running test without leaving gigabyte dumps behind.
  `--blame-crash` would need a dump tool and space.
- The env line is reported in the gate's output rather than in the darn prompt.
  That way it reaches every caller of a tier, not only darn writers.
- The measuring call keeps the `darn-write` key that the Darn-Write fixer wiring already uses.
- The red-first gate is waived for this change, as the decision record's "No test seam" entry records:
  the diff changes the darn stencil `darn-fragment-make-change-gated.md`, a non-Go file the red-first overlay cannot restore on the merge base.

## Open risks

- A `vstest` tier whose command holds `{args}` but runs `dotnet vstest`, or `dotnet test` in Microsoft.Testing.Platform mode, receives argument spellings it does not take.
  Its command fails, which points to the cause.
- The env line prints both values.
  A tier `env` that overrides a secret the session holds would show that secret in the gate output and log.

## Merge danger

**Door:** two-way
**Blast radius:** gate
