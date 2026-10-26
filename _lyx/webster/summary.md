# Beam start and ownership: all 14 batches done

Every batch in the card list reached `status: done`.
Batches 01, 02, 04 and 06 were already done when this session resumed; this session ran the other ten.
No batch needed `recover-batch`, no digest carried `departures` or `renames`, and no `warnings` came back.

## Batches run this session

- **07 dobby-start-named-beams** (cards 7, 8, head `91b77c1a`): `EnsureLive` starts only named beams and skips a beam its landing holds, and the orch's watcher stops the orch of a beam another machine owns.
  Deviations: `internal/adoptcli/adopt_test.go`, `internal/dobbyengine/start_test.go`, `internal/fabriccli/anchor.go`, `internal/fabriccli/anchor_test.go`, `internal/hubgeom/dobbygeom.go`, `internal/orchengine/owner.go`.
- **09 remove-orch-cleanup** (card 9, head `413efefd`): `lyx orch cleanup` is removed.
  No deviations.
- **10 orch-beam-up** (card 10, head `d4f81ec1`): `orchcli.BeamUp` and `BeamDown` replace `lyx orch start`, resuming first, then the last note, then the stencil.
  Deviations: `internal/orchcli/cli_test.go`, `ownerpause.go`, `smoke_cycle_test.go`, `start.go`, `start_test.go`, `stop.go` and `internal/orchengine/launch.go`.
- **11 beam-start-stop-status** (cards 11–14, head `c02278b3`): `lyx beam start [slug] [--all]`, `lyx beam stop`, the orch in `lyx beam status`, the `lyx s` shortcut, `BeamLandRefusal` judging members and pairs alone, and Finalize's re-check of a beam's members under the parent lock.
  Deviations: `internal/beamcli/cli.go`, `start.go`, `target.go` and `internal/landingshed/finalize_land.go`.
- **15 pawl-engine** (card 15, head `3dc0cb4b`): the new `internal/pawlengine` package with the Beam-Up, Beam-Await and Beam-Down producers and the land request.
  Deviations: files inside the new `internal/pawlengine` package only.
- **16 pawl-recipe-rows** (cards 16, 17, head `4254ddd6`): the pawl recipe's nine rows, registry entries, budgets, profiles and the loomcli wiring of the seams and Finalize's re-check.
  Deviations: `internal/loomcli/pawlwiring.go`, `sharedbootstrap.go`, `internal/loomrecipe/pawlrecipe_test.go`, `internal/runprofile/runprofile_test.go`, `internal/shedrecipe/entries_pawl.go`, `fixture_test.go`, `seam_enforcement_test.go`.
- **18 batten-beam-project** (cards 18–21, head `887859a0`): batten starts a beam project's run, no quiet notice for a child at Beam-Await, a beam's cancel reconciles and stops its orch, and `lyx beam create` starts the pawl run.
  Deviations: files under `internal/battencli` and `internal/beamcli`.
- **22 beam-land-request** (cards 22–24, head `a69891b0`): `lyx beam land` writes a land request, the dobby posts the "can land" notice, and `lyx adopt` brings up each adopted beam's dobby and orch.
  Deviations: files under `internal/adoptcli`, `internal/beamcli` and `internal/dobbyengine`.
- **25 adopt-subject-ways-forward** (card 25, head `75ff02a0`): owner ways forward name the beam or `root`, never a task.
  Deviations: files under `internal/battencli`, `battenshed`, `dobbyengine`, `fabriccli`, `fabricengine`, `landingshed`, plus `internal/testkit/envkit/envkit.go`.
- **26 name-beam-start-everywhere** (card 26, head `3174135c`): every way forward names `lyx beam start` where it named `lyx orch start`, and `TestRemovedVerbs_NamedNowhere` scans for the removed verbs.
  Deviations: `cmd/lyx/removedverbs_test.go`.

## Verify gate findings

The first plan-level verify failed on two identities, both from the findings file:

- `github.com/Knatte18/loomyard/cmd/lyx` (package).
- `github.com/Knatte18/loomyard/internal/boardcli.TestCLI/Profiles` (test).

A fixer fork was spawned for them.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `72edea22c07fb560b4bc0e487033ef1af2ec895d` fix: the pawl profile default in the boardcli test and the batten cancel Long text under its cap


## Plan rebaselined

webster: the plan changed since the run recorded it and was rebaselined on entry, keeping 4 begun batch(es) and accepting 10-orch-beam-up.md, 12-shortcut-s.md, 14-finalize-land-recheck.md, 15-pawl-engine.md, 16-pawl-recipe-rows.md
