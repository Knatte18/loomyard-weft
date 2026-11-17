# Machine registry, Bolt holds and catch-up from origin

All nine batches recorded `done`, each by its fork's first report, with no recovery, re-fork or plan fixer.
No digest listed a Pattern-Departure or a rename.
Every digest's `deviations` list is informational; the files named below are the scope guard's result.

- **01-fixtries-package** (head `fc1779cf`): `internal/fixtries` with `Read` and `Write`, `machine.json` identity (id, forgotten names), and Bolt's `machines` and `holds` records with the registry comparison.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/boardengine/registry.go` and its test, `internal/fixtries` files, `internal/machinename` files.
- **04-machine-registration-setter** (head `a16a129c`): the registry setter with its refusals, the machine id write, the synchronous hold write, and registration in every owner write.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/boardengine/holdwrite.go`, `identity.go`, `registrysetter.go`, `internal/testkit/tmuxkit/tmuxkit.go` and its test.
- **07-owned-elsewhere-through-registry** (head `73db3dd1`): `OwnedElsewhere` takes the registry; name comparisons moved onto it.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/adoptcli/adopt.go`, `internal/battencli/owner.go`, `internal/beamcli/create.go` and `land.go`, `internal/loomcli` ownercheck test and `pawlwiring.go`, `internal/orchcli/refusal.go` and `start_test.go`, `internal/pairteardown` `originseams.go`, its test, and `runread.go`.
- **08-dobby-start-owner-flags** (head `66101616`): owner flags from the registry, the registry-derived name set, `lym machine name` registering the rename, and `--forget-former`.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/boardengine/identity.go`, `internal/dobbyengine/start.go`, `internal/machinecli` (`cli.go`, `forget.go`, `set.go`, `set_test.go`), `internal/machinename/machinename.go`, `internal/orchcli/start.go`.
- **12-bolt-hold-caller** (head `48c29e06`): the bolt-hold caller, `ReadRun` filling `Held` from Bolt, Bolt fetch before the act rule, tender bolt-hold entries, no queued task claimed under a hold, hand-over of a run another machine owns.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/battencli/dobbyreading_test.go`, `internal/dobbyengine` (`starts.go`, `view.go`), `internal/pairteardown` (`handover.go`, its integration test, `runread.go`), `internal/pauserecord/record.go`, `internal/tendercli` (`cli.go`, `runintents.go`), `internal/tenderengine` (`bolthold.go`, `pause_test.go`).
- **18-catch-up-engine** (head `b2c3b433`): `dobbyengine.CatchUp` over seams, pause entries from intents, the run's own pass catching up with the origin-owed marker retired, and the dobby catching up after every absence.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/battencli` (`arm.go`, `catchup.go`, its integration test, `owner.go`, `readopt.go`), `internal/beamcli/dobbydeps.go`, `internal/dobbyengine` (catch-up, hand-over, loop and pass files and tests), `internal/tendercli` (`pauseintents_test.go`, `runintents.go`).
- **22-upgrade-first-read** (head `48077599`): standing adopt entries become pause intents and one Bolt hold, run at the dobby start before the catch-up.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/beamcli/dobbydeps.go`, `internal/dobbyengine/upgrade.go` and its test.
- **23-retire-owner-pause-callers** (head `216e1bd6`): no code path writes the owner caller; the bring-up ends owned-elsewhere runs through the `endOwned` and `isBeam` seams per the parent's recorded decision.
  Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/battenshed` (`innerrun_owner.go`, `innerrun_parentlock_test.go`), `internal/dobbyengine` (`passes_test.go`, `starts_test.go`), `internal/hibernateengine/world_test.go`, `internal/loomcli/prepush.go`, `internal/machinecli/cli.go`, `internal/orchcli/start.go`.
- **24-delete-tender-owner-pause** (head `d86ff585`): `PauseOwnedElsewhere` and its process end deleted; `PATTERN-owner-machine`'s changed line and background written.
  Deviations: `cmd/lym/testonlycode_linux_test.go` only.

Fork reports flagged points for review, not failures:
- `dobbyTaskWait` in `internal/battencli/overviewdobby.go` gives no reason for a task held by an adopt hold, so the text view of `lym beam status` can print "waits on" with nothing after it.
- The one-time identity-refusal post to the prime's orch in `internal/orchcli/start.go` has no test.
- `Drop` in the upgrade's first read goes through tender's `Lift`; whether `Lift` also sends a resume line before the catch-up re-pauses was not checked.
- An error from `Classify` fails the upgrade's whole read, so a standing entry for a slug that has left the board could keep the dobby's catch-up failing.
- The card gates and batch gates ran tiers 0 and 1; tiers 2 and 3 did not run.
