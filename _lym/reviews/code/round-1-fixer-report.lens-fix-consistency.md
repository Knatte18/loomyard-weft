```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked whether the F1–F5 change set still agrees with its surroundings and found one MEDIUM and one LOW inconsistency.

**MEDIUM — the PATTERN exemption list is incomplete.**
- **Changed text:** `pattern/PATTERN-child-processes.md`, the new sentence "The drive wiring in `internal/loomcli` and the wire in `internal/battencli` also call it directly: each resolves the path once at wiring."
- **Problem:** the amended rule still doesn't match the code. Two more sites call `os.Executable` directly and hand the path to a spawner, and the rule names neither as an exemption:
  - `internal/hubgeom/slackgeom.go:46` and `internal/hubgeom/dobbygeom.go:34` put `os.Executable()` into `LymExecutable`. `internal/slackengine/start.go:95` and `internal/dobbyengine/start.go:299` run it with `exec.Command(geom.LymExecutable, ...)`. That is a detached lym child, which the rule says must resolve through `selfexec.LymBinary`.
  - The review's F1 asked for the rule to match the code. The review's list of sites did not name these two, but they are the same class as the five it did name.
- **Fix:** either convert the two geometry builders to `selfexec.LymBinary`, or name them in the exemption.

**LOW — two comments still describe the old call.**
- `internal/reedcli/cli.go:42` says "Spawning os.Executable() from a test binary would run the whole suite recursively".
- `internal/reedcli/remove.go:58` says "Re-exec'ing os.Executable() there would run the whole suite".
- Both are now stale next to `selfexec.LymBinary`. The `testing.Testing()` and suppress guards still hold, so only the wording is stale.

I checked these and found them consistent:
- The `tenderengine` `Executable`, `adoptcli` `runningLymBuild`, `statuspane` `CurrentBuild` and `stencilcommit` `BinaryLabel` sites only compare or read the binary, so they fit the PATTERN's read-only exception.
- The `reedengine` `doc.go` text agrees with `SpawnWatchdog` calling `selfexec.LymBinary` and with `buttonclick.go` now using `ResolveLymBinary`.
- The `reedcli` window seam name `lymBinary` matches `loomCLI.lymBinary`.
- The `ideengine` `LymBinary` seam is set in `testmain_test.go`.
- The `tmuxkit` `Main` doc covers the run-directory removal.

Calls made:
1. Grepped `.Executable` in `internal/slackengine`, `internal/dobbyengine` and `internal/tenderengine` to see how the `hubgeom` and tender executable paths are used. My context did not show their consumers.
2. Read `internal/reedengine/spawnwatchdog.go` lines 1–30 and grepped `os.Executable` in `internal/reedcli/cli.go` and `internal/reedcli/spawnwatchdog.go` to check the watchdog claim and stale comments. My context had not read `spawnwatchdog.go` or the `reedcli` comments.
3. Grepped `Executable` in `internal/slackengine` and `internal/dobbyengine` to confirm the spawn use. The first grep only showed the tender hits.
