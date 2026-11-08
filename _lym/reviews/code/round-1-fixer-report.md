# Fixer report, code round 1

All five findings are fixed, each in its own commit, and the touched packages pass `lym gate test` at tier 0 and tier 1.

## Changes

- **F1** (c31e547b, plus a follow-up commit)
  - `internal/creelcli/spawn.go`, `internal/gatecli/detachfront.go` and `internal/reedcli/remove.go` now resolve the binary through `selfexec.LymBinary`, keeping their `testing.Testing()` and suppress guards.
  - `internal/boardengine/spawn.go` defaults `syncExecutable` to `selfexec.LymBinary`.
  - `internal/reedcli/window.go` resolves through a new `reedCLI.lymBinary` seam that defaults to `selfexec.LymBinary`; `window_test.go` injects a stand-in and asserts the launcher receives it.
  - `internal/ideengine/spawn.go` stamps the vscode config path from a new exported `LymBinary` var, which defaults to `selfexec.LymBinary`. The package's `TestMain` points it at an absolute stand-in, the way it already does for `KeybindingsPath`.
  - `PATTERN-child-processes.md` now names the exemptions: the `internal/loomcli` drive wiring and the `internal/battencli` wire each resolve once at wiring, and the `internal/hubgeom` geometry builders hand the path to spawners that refuse under `go test`.
  - Two stale `os.Executable()` comments in `internal/reedcli` were reworded.
- **F2** (b0e46307): `internal/reedengine/buttonclick.go` wraps `deps.executablePath` in `selfexec.ResolveLymBinary`. The `internal/reedengine/doc.go` sentence now also says `SpawnWatchdog` calls `selfexec.LymBinary` directly.
- **F3** (7ef89520): `testRequest.processCapFor` is a new seam over `newProcessCap`, defaulting to it. `TestRunTierProcessCap` covers a tripped cap and an unmarked run:
  - A command that exits 0 under a tripped cap gives exit 1 and sets `processCapped`.
  - The cap line reaches the causes and stderr.
  - `failedOnShortage` returns empty for the capped result, though its log holds a shortage marker.
  - A run whose environment has no run marker never starts the cap.
- **F4** (ca810f20): `tmuxkit.Main` removes the process run directory before it returns the refusal exit. The package's `TestMain` makes that directory first, and the integration test points the child at a private `TMPDIR` and asserts nothing is left in it.
- **F5** (aaa5c6bf):
  - The `TestBinaryExecutable` row of `internal/reedengine/bindings_test.go` now also asserts the follow line and the M-Left and M-Right bindings.
  - `TestComposeLines_DropThePreludeForTheTestBinary` covers the pane launch line and the window command.
  - `internal/loomcli/cli_test.go` gets a lookup-failure subtest for `loopExecutable`.
  - `internal/fronttask/hub_test.go` gets a lookup-failure row for `startAction`.

## Gate repair

The gate refused three comments the round added for wrapping mid-sentence.
They are now one sentence per line:
- the `TestMain` comment in `internal/ideengine/testmain_test.go`,
- the lookup sentence in `internal/reedengine/doc.go`,
- the test comment in `internal/reedengine/panebin_test.go`.

No code changed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, MEDIUM: the `PATTERN` exemption list omitted `hubgeom`'s slack and dobby geometry builders. Fixed by naming them in the exemption; their spawners (`slackengine`, `dobbyengine`) refuse under `testing.Testing()`, which is the exemption the review allowed. Converting them would make the geometry builders fail under every test binary.
- fix-consistency, LOW: two stale `os.Executable()` comments in `internal/reedcli`. Fixed.
- fix-proposal, MEDIUM: same `hubgeom` finding, merged with the one above. Fixed the same way.
- fix-proposal, LOW: callers of `ideengine` outside the package might stamp the bare name under test. Checked: `internal/idecli` and the other packages passed at tier 0 and tier 1, and no test asserts an absolute stamp outside `ideengine`. No change.
- fix-proposal, LOW: the resolve-once wording gives no verifiable reason for `loomcli`. Disputed, `premise false`: the amendment claims only that each site resolves the path once at wiring, which the review named and the code shows (`drivewiring.go:171`, `wire.go:1039`).
- fix-affected-set, LOW (two entries): the same `hubgeom` finding, and `tendercli` and `adoptcli` as reads of the binary itself. The `hubgeom` one is fixed as above; the other two stay on `os.Executable` as reads, which the rule allows.

All four forks returned a report.

## Disputed

- Fork finding, fix-proposal LOW 3 (resolve-once wording), ground `premise false`. Evidence is in the Self-check entry above.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.
