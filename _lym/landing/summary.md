# Add lym machine name to show and set the machine name

`lym machine name` shows this machine's name and sets a new one.
A rename now also moves the owners recorded under the machine's former names to the new name, so open work stays owned across it.
Until now the name came from the machine file or the hostname with no verb to change it, and a renamed machine read its own entries as another machine's.

## What changes

- New module `machine`, package `internal/machinecli`, wired under the root as a cobra subtree (`PATTERN-cli-cobra`).
  It runs from any directory, inside a hub or not.
- With no argument the verb prints the name, its source (`file` or `hostname`), the file's path and the former names, and writes nothing.
  A bad file prints the file's error and path instead of a name.
- With a name it validates the name, then writes the machine file through `internal/state`:
  `name` becomes the new name, the old name and the hostname rule's name are appended to `names` when absent,
  empty entries are dropped (the output says so), and the new name moves to the end of `names`.
  It refuses to write when no config directory resolves or a variable is relative.
- Before writing, it pulls the Bolt of every hub it will sweep and refuses a name that a board records as an owner
  when the name is not one of this machine's names.
  A hub whose pull fails or skips its fetch is checked against its local Bolt, and the output says so.
- A rename sweeps every hub `hubregistry.Live` lists, plus the hub of the working directory.
  Each hub gets one `boardengine.SetOwners` call.
  A hub whose write fails or returns `OwnerWrite.Lost` entries is reported with a re-run of `lym machine name <name>` as the way forward,
  the other hubs go on, and the verb exits non-zero.
  A re-run with the name already set rewrites the file only when `names` changes, and sweeps again.
- Over a bad file the verb reads the file itself through the new `machinename.ReadFile` path:
  a file that decodes keeps its `names` and drops its invalid `name`;
  a file that does not decode is replaced and its names are lost, which the output says.
- `boardengine.OwnerRequest` gains `Replace`, the names whose recorded owners the write takes over.
  After the write's own pull it selects the prime and every open beam project whose recorded owner is in the set,
  so an owner only origin holds is swept too.
  It goes with no `Slugs`, `Prime`, `Claim` or `Takeover`; `SetOwners` refuses the combination.
  Done entries and owners recorded under other names are untouched, and empty entries select nothing.
- `machinename.FromThisHost` returns the hostname rule's name without reading the machine file.
- `PATTERN-owner-machine` and `PATTERN-cli-cobra`, `docs/overview.md` and the `lym --help` module list are updated.

## Design decisions

- Owner checks compare strings, so between the file write and a hub's sweep that hub's owners read as another machine's.
  The verb's output names each recovery: the dobby's probe lifts its own owner pause after the sweep,
  `lym tender resume <slug>` lifts one it does not, `lym loom resume` restarts a halted Finalize,
  and `lym beam start <slug>` (or `root`) restarts a worktree skipped as owned elsewhere.
  It advises lifting a pause only after the hub's sweep reports done, since a lift skips the deferred push of pairs it reads as owned elsewhere.
- A hub that is neither in the registry nor holds the working directory is not swept; run the verb from inside it.
- A name shared with another machine passes the collision check when that machine has recorded no owner on a swept board,
  or when it is a former name or the hostname rule's name.
  The way forward is renaming one machine to a name no other holds, then `lym adopt` for entries the other's sweep moved.
- Tests reach the machine file and hubs only through `machinecli`'s deps, so none writes the developer's machine file or a real hub's Bolt.

## Also in this diff

Two unrelated test and logging fixes:

- `internal/sheddriver` logs the loop's stop detail when it hands a run back (`Stop.Detail`).
- `TestSmokeBootstrapLifecycle` in `internal/loomcli` clears a killed loop's pid file and envelope before the rigged run.

## To check by hand

- Run `lym machine name` with no argument on a real machine, then rename and re-run against a hub with an unreachable origin;
  no test exercises a real remote.
- The final integration gate failed once and passed on rerun for two tests that triage did not attribute to this change:
  `internal/gatecli.TestGateTest_DetachedRun/a_passed_lease_ends_a_--wait_gate_line_through_its_child,_which_kills_its_tiers_and_frees_its_slots`
  and `internal/gatecli.TestRedundancyStep/ABrokenLymBuildAndACorruptMapAreErrorLinesThatLeaveTheOtherTestsJudged`.
  Treat both as flaky.
