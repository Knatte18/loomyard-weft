# Beam groundwork

All nine batches recorded `status: done`, and no digest carried a `departures` list.
Every digest listed `deviations`, files changed outside a batch's own card targets; each was informational, and none failed a batch.

- **01-fabric-slug-rule** (head `6b98a7a26`): exported the pair slug rule, added the beam mark to the origin record, made `AddOptions` its own struct and let `Add` fork from a named parent pair.
  Deviations: `internal/fabricengine/origin.go`, two new integration tests, and `internal/hubforge/hub.go`, which needed a `SyncOptions` built once `AddOptions` changed.
- **05-add-adopt-local** (head `9e3d5f121`): `Add` over an existing local branch, `--parent` and `--adopt-local` on `lyx fabric add`, a workspace file per beam, and `ResetRecordsToParent`.
  Deviations: files in `internal/fabricengine` (`add.go`, `addadopt.go`, `destroy.go`, the hub workspace files and tests).
- **09-board-project-kind** (head `a8e409463`): the board's project kind, membership rules, owner-setter beam flag, `ParentBeam` and the slug rule.
  Deviations: supporting files in `internal/boardengine` (`runlock.go`, `store.go`, `task.go`, `layer.go` and their tests).
- **14-board-readme-projects** (head `dd39317f3`): the README Projects section, `hubgeom.ParentOf` for every owner reader, removal of `fabricengine.ParentSlug`, `lyx board` wiring and `NearestLiveOrch`.
  Deviations: `internal/boardcli/cli_test.go`, `internal/boardengine/render.go`, `internal/hubgeom/nearestlive.go`.
  `PATTERN-pair-parent` was added to PATTERN, as declared.
- **19-orch-in-beam** (head `94adcaf7e`): `lyx orch` in the prime and any beam, `NotifyParent`, and tender, hibernate and the watcher reading every orch.
  Deviations: `internal/orchcli`, `internal/orchengine/pause.go`, `internal/tendercli/deps.go`, `internal/tenderengine` and `internal/hibernateengine/verbs_test.go`.
- **22-overview-cleanup-every-orch** (head `35b54ed1b`): overview and cleanup over every orch, batten, adopt and cleanup from any beam, the window opened in the parent's session and notices routed up the beam chain.
  Deviations: `internal/battencli`, `internal/adoptcli`, `internal/cleanupcli/segments.go`, `internal/orchcli/live.go` and its test.
- **26-land-ready-routing** (head `01065e17c`): ready notices routed by destination, the cancel named in cleanup's skip, a beam member's pair forked from the beam, a large merge gated by Finalize, and the beam description producer.
  Deviations: `internal/battencli`, `internal/battenshed` and `internal/landingshed/beamdescribe.go`.
- **31-beam-recipe** (head `bf31222b2`): the beam recipe and its registered row, loom's bootstrap running it, and Seed-Child seeding it for a project entry.
  Deviations: `internal/loomcli`, `internal/loomrecipe`, `internal/shedrecipe/conformance_test.go`, `internal/testkit/envkit/envkit.go` and `internal/battencli/wire_seams_test.go`.
- **34-beam-create** (head `1b6c17458`): the `lyx beam` module with `create`, `promote` and `land`, and `lyx adopt` taking over a beam.
  Deviations: `cmd/lyx/sandbox_coverage_test.go`, `internal/adoptcli/cli.go`, `internal/battencli`, `internal/beamcli` files and `internal/boardcli/cli.go`.

Scope guard: no batch failed on its deviations.

## Verify gate findings

The first verify gate attempt failed in `internal/boardcli` and in three `internal/testkit/envkit` tests: `TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths` and `TestNilSeams_SkipsNilLegalPaths`.
A fixer fork handled them.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `c324976d420f6b5575dc4ac2c31036eb5b6b1e2c` fix: restore the upsert help's notice line within the cap and make Land.ReadyRoutes nil-legal in envkit
