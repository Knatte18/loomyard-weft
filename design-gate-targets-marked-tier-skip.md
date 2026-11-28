# gate test --gate K with explicit targets runs a K-marked tier1 test under tier0, where it skips with exit 0

`gate-targets-marked-tier-skip` · Note · bug, gate

Imported from [issue #298](https://github.com/Knatte18/loomyard/issues/298).

## What happened

A darn writer had to time a tier1 test marked `//lym:only upkeep`, before and after its change, through `lym gate test`.

- The gate key its prompt gives it, `darn-write`, skips the test because of the marker, so getting the timings took a round trip to the parent.
- `lym gate test --gate upkeep <package> -- -run <test>` then ran the package under tier0, where the tier1 test skipped, and exited 0.
- The same call with no package argument ran the test under tier1.
- `--tier 1` beside `--gate upkeep` is refused.

## Why it is abnormal

The call that names the package and the test reads as a successful run, yet the selected test never ran, so a timing taken from it is wrong without any failure to show it.
The behaviour follows from two rules that are each documented: explicit targets beside `--gate` replace the key's scope, and a `marked` key runs its tests under each file's tier only in its own scope.
The writer saw exit 0 and no refusal.

## What to do

- With `--gate K` and explicit targets, run the tests marked for `K` in those packages under their files' tiers, or refuse the call and name the form that runs them.
- At least, say on stderr that a test marked for the key belongs to a tier this call does not run.
- Give the darn writer's measuring call a stated way to reach a test held by another key's marker, such as naming `--tier` with a `--filter` and no `--gate`, which holds nothing back.

Provenance: task `gatecli-dotnet-redfirst-time`, note `Darn-Write.md`.
