# Red-first gate passes a hub whose scoper is not go

Fixes [issue #192](https://github.com/Knatte18/loomyard/issues/192).

On a hub whose test scoper is not go, such as a C# hub, `lym gate red-first` cannot run the diff's tests on the merge base and exits `gateslot.RedFirstUnjudgedExit`.
The Darn-Write red-first gate treated that exit as a hold only the parent's `No test seam` record lifts,
so every bug-labelled run on such a hub stopped until the parent recorded a manual waiver.

The gate now passes that exit and logs that it passed unjudged.
A timeout or runner crash that exits with the same code still fails terminally,
and a diff that adds or changes no test still holds the run as before.

## Changes

- `internal/loomshed/redfirstgate.go`: `verify` returns a passing verdict for `RedFirstUnjudgedExit`;
  the hold and failure paths no longer handle that exit.
- `internal/gatecli/redfirst.go`: the verb keeps exit 5, and its message now says a red-first gate passes it instead of asking for a `No test seam` record.
- `contracts/stencils/darn/darn-template-write.md`: the darn writer's bug-entry passage says the gate passes on a non-go hub, and no longer tells the writer to ask for a waiver there.
- Package docs of `loomshed`, `shedrecipe`, `gatecli` and the `gateslot.RedFirstUnjudgedExit` doc follow.

## Design decisions

- The gate skips red-first rather than running the hub's own test command on the base.
  For a non-go scoper the verb has no way to put the base's production code back under the working tree's tests:
  the Go build overlay it uses has no counterpart for other toolchains.
- The skip lives in the gate site, not the verb: the verb's exit 5 still means "could not judge", so a hand run reports what happened.

## Tests

- `TestRedFirstGate_VerdictPerLabelRecordAndExitCode`: the exit-5 rows, for an arrival and a final evaluation, now pass with no record; on the merge base they held and failed.
- `TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands` drives its hold with exit 4, the no-test refusal, since exit 5 no longer holds.
- `TestGateRedFirst_AHubWhoseScoperIsNotGoExitsUnjudged` asserts the verb's new message.

## Merge danger

**Door:** two-way
**Blast radius:** loomshed
