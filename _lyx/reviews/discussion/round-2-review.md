---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D1
    summary: help becomes a group with an index child, but D1 never settles how it meets the clitree walk's bare-group listing and unknown-subcommand refusal, which its kept cobra behaviour breaks
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: the watcher's PATH lookup resolves the watcher's inherited PATH, not the orch pane's prelude PATH, so it can describe a different binary than the one the orch calls
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D1
    summary: D1 refuses an unknown topic but leaves open a known group followed by an unknown word, which cobra's Find resolves to the group's help with no error
---

### [BLOCKING] `help` as a group conflicts with the test-enforced group invariant

**Location:** `_lyx/discussion/decision-record.md` D1, with Notes for the plan writer on `walkCommands`.

**Issue:** D1 makes `help` a module whose child is `index`, so `help` is a group.
`PATTERN-cli-cobra` says one walk of `newRoot()` in `cmd/lyx/clitree_test.go` enforces bare-group listing and unknown-subcommand refusal for every command.
`TestCLITree_EveryCommand` runs each walked group bare and requires exit 0 and output naming every visible child (`bareProblems`).
It also runs `<group> bogus` and requires exit 1 with an error containing `unknown subcommand` (`bogusProblems`).
D1's bound keeps cobra's behaviour, and cobra's help command with no argument prints the root's help.
That output lists the modules, not `index`, so it fails the bare check.
D1 also refuses an unknown topic with "a JSON error naming the topic", which does not say it contains `unknown subcommand`.
`walkCommands` skips `help` by name today, and the Notes say only that "the skip must let the new `help` module's `index` through".
So the record leaves open whether `help` itself stays exempt from the walk and why, or what bare `lyx help` prints so that it names `index`.
D3's audience test and D6's form test run over the same walk, so the same choice decides whether `help` (a runnable group by D3's definition, since `help [command]` names a positional) and `index` carry an audience and pass the `Use` form.

**Fix:** In D1, state how `help` meets the walk.
Either bare `lyx help` prints output naming `index` and an unknown topic's error contains `unknown subcommand`, or `help` is exempted from the bare and bogus checks by name with the reason, while the audience and form tests still reach `help` and `index`.
Add an acceptance criterion for the choice, and update the Notes entry on `walkCommands` to match.

### [BLOCKING] The watcher's `lyx` on `PATH` is not the orch pane's `lyx`

**Location:** `_lyx/discussion/decision-record.md` D4, A9, the Constraints line on `PATTERN-spawn-observability`, and the support log's Rejected alternatives on the watcher.

**Issue:** D4 says the index describes "the `lyx` the orch's pane resolves on `PATH`", and the watcher gets it by running `lyx help index` "through the `lyx` on `PATH`".
These are two different `PATH`s.
The orch's pane resolves `lyx` through reed's pane prelude (`internal/reedengine/panebin.go`), which prepends the spawning binary's directory to the pane's `PATH` and exports `LYX_BIN`.
The `resume-context` hook runs inside the orch's provider session, so it inherits that prelude.
The watcher does not.
`spawnWatcherProcess` in `internal/orchcli/spawn.go` starts `os.Executable()` with `orch watch` and the environment of the `lyx orch start` process, never the prelude.
When the orch is started from a binary that is not first on the operator's `PATH`, such as `.dev-bin/lyx` from `./deploy-dev`, the watcher's lookup finds a different `lyx` (or none).
Its reload renders would then describe one binary while start, adopt and `resume-context` describe another, writing alternately into the same role file.
`PATTERN-pane-binary-resolution`'s background also excludes the detached daemon spawns because "neither resolves `lyx` from `PATH`"; D4 adds a daemon that does, without saying so.

**Fix:** In D4, name the binary as the orch pane's resolution: an absolute path the watcher is told, the one `lyx orch start` or `--adopt` ran from (the path the pane's prelude exports as `LYX_BIN`).
A deploy that rebuilds that same path is then picked up at the next render.
State the bound when that path no longer exists, and update A9, the Constraints line and the support log's rejected alternative to match.

### [LOW] An unknown word under a known group is not settled

**Location:** `_lyx/discussion/decision-record.md` D1, Bound paragraph, and the acceptance criterion on `lyx help no-such-topic`.

**Issue:** D1 refuses an unknown topic with a JSON error.
Cobra's help command resolves the topic with `c.Root().Find(args)`, which returns an error for an unknown word only at the root.
`lyx help board bogus` finds `board` with no error, so cobra prints the board group's help.
D1 does not say whether the replacement keeps that or refuses `bogus`.

**Fix:** State in D1 which way `lyx help <group> <unknown>` goes, and cover it in the acceptance criterion beside `lyx help no-such-topic`.

## Focus directives judged with no finding

- D7 and the merge entry: the coupling test compares required keys, optional keys and exclusive pairs as three sets, nested objects included.
  `merge {remove_slugs?, upsert: <task>, set_status?: {slug|id, status}}` matches `internal/boardcli/cli.go`: it refuses keys other than `remove_slugs`, `upsert` and `set_status`, refuses a missing `upsert`, and validates `set_status` through `resolveLookup` with `status` required.
  Not a defect.
- D2 and D3 on runnable groups: D3 defines a runnable group as a group with its own run function whose `Use` names a positional or that has local non-hidden flags, and names `lyx config [module]` with `--print` and `--set` as one.
  `configcli.Command` matches that, so `config` carries an audience and gets a leaf-form line (D2, and the acceptance criterion on `[<module>]`).
  Not a defect.
- D2, D3 and D1 on `completion` and unannotated leaves: D2 skips the `completion` subtree and prints no command without an `audience`, and D1 keeps `completion` out of `lyx --json`.
  D1 keeps argument completion for `lyx help <tab>`.
  Settled; the unsettled part of D1's help-command bound is in F1 and F3.
