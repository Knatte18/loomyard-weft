# Gate tier timeout leaves out suspended time, and --gate with explicit targets runs or refuses marked tests

`gate-suspend-marked-targets` · Task · bug, gate, hibernate · draft

Depends on: [`card-gate-own-packages`](design-card-gate-own-packages.md)

Two gate bugs from intake, bundled because both live in the gate's tier run and target selection.

## Tier timeout counts hibernation (issue #295)

The machine hibernated from 2026-11-27 22:47:54 to 2026-11-28 08:52:17 while `lym gate` was running tier1.
On wake, the gate killed tier1 as past its 30-minute limit.
The timeout report asked the reader to find a hang or to raise `gate.tiers.tier1.timeout_min`.
Neither applied, and only the system journal showed the cause.

The tier timeout counts wall-clock time across a hibernation, so time in which the tier could not run is charged against its limit.
A run that was not hung fails, and the report sends the reader after a cause that is not there.

What to do:

- Exclude suspended time from the tier timeout.
- Or detect a suspend during the tier run, say so in the report, and rerun the tier instead of reporting a timeout.

Provenance: task `discussed-payload-map-race`, note `gate-fixer-darn-write.md`.

## `--gate K` with explicit targets runs a K-marked tier1 test under tier0 (issue #298)

A darn writer had to time a tier1 test marked `//lym:only upkeep`, before and after its change, through `lym gate test`.

- The gate key its prompt gives it, `darn-write`, skips the test because of the marker, so getting the timings took a round trip to the parent.
- `lym gate test --gate upkeep <package> -- -run <test>` then ran the package under tier0, where the tier1 test skipped, and exited 0.
- The same call with no package argument ran the test under tier1.
- `--tier 1` beside `--gate upkeep` is refused.

The call that names the package and the test reads as a successful run, yet the selected test never ran, so a timing taken from it is wrong without any failure to show it.
The behaviour follows from two rules that are each documented: explicit targets beside `--gate` replace the key's scope, and a `marked` key runs its tests under each file's tier only in its own scope.
The writer saw exit 0 and no refusal.

What to do:

- With `--gate K` and explicit targets, run the tests marked for `K` in those packages under their files' tiers, or refuse the call and name the form that runs them.
- At least, say on stderr that a test marked for the key belongs to a tier this call does not run.
- Give the darn writer's measuring call a stated way to reach a test held by another key's marker, such as naming `--tier` with a `--filter` and no `--gate`, which holds nothing back.

Provenance: task `gatecli-dotnet-redfirst-time`, note `Darn-Write.md`.
