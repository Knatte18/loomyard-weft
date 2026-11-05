# pawl-orch-landing: beam landing moves into Beam-Await

All seven batches (cards 1–18) recorded `done`; the last head is `ef47f3e32050a53b6fec07182d44faffcf05db4e`.
No batch needed recovery, no digest carried `departures`, `renames` or `warnings`, and no friction note was written.

## Batches

- **01 shed-retired-rows (cards 1–4):** shed moves a run off a recipe-declared retired row and pauses it; the tracked beam landing record sits beside the ready record; `ReadMergeState` reads a pull request's merge state and required checks, with the shared `prkit` fake; the beam description writer is a function.
  Deviations: `cmd/lym/ghguard_test.go`, `internal/landingshed/beamdescribe.go`, `internal/loomengine/config.go`, `internal/pawlengine/landing.go` and its test, `internal/pullrequest/client.go`, `client_test.go`, `mergestate.go` and `mergestate_test.go`, `internal/shedbuild/parse.go` and its test, `internal/shedengine/retired_test.go`, `internal/testkit/prkit/prkit.go`.
- **05 pawl-rows (card 5):** pawl's six rows, the Finalize hold, and the BeamDescribe producer's removal.
  Deviations: `internal/loomcli/peers.go`, `peers_test.go`, `wiring_darn_test.go`, `internal/loomrecipe/pawlrecipe_test.go`.
- **06 remove-reopen (card 6):** `lym beam reopen`, the reopen request and PR-Review's reroute are removed.
  Deviations: `internal/landingshed/prreview.go`, `internal/loomrecipe/pawlrecipe_test.go`, `internal/pawlengine/beamawait_test.go`, `landrequest.go`.
- **07 ready-beam-starts-no-member (cards 7–9):** a ready beam starts no member; pawl's side effects of a move off a retired row; re-invoke policy for the pawl Go rows and the completed-wake repair rule.
  Deviations: `internal/dobbyengine/loop.go`, `starts.go`, `internal/loomcli/pawlwiring.go`, `wiring.go`, `internal/pawlengine/retiredmove.go` and its test, `internal/shedrun/wake.go`.
- **10 beam-await-entry (cards 10–11):** Beam-Await settles a landing hold on entry and stamps its wakes; Finalize lands a beam by its landing record and keeps its pull request's text.
  Deviations: `internal/landingshed/finalize_pr.go`, `internal/loomcli/landingdeps.go`, `internal/pawlengine/beamawait.go`, `world_test.go`.
- **12 beam-describe-verb (cards 12–13):** `lym beam describe`, `lym beam pr open` and `update`.
  Deviations: `cmd/lym/ghguard_test.go`, `internal/beamcli/cli.go`, `land_test.go`, `pr.go`, `pr_test.go`, `internal/fabricengine/beamline_integration_test.go`.
- **14 beam-pr-status-close (cards 14–18):** `lym beam pr status` and `close`; `lym beam ready` refuses what the orch can still fix; the dobby announces member landings while a pull request is recorded; `lym beam start` brings back the orch of a beam in landing; the orch role text carries the landing workflow.
  Deviations: `internal/beamcli/cli.go`, `dobby_test.go`, `dobbydeps.go`, `land.go`, `land_test.go`, `pr.go`, `pr_test.go`, `internal/dobbyengine/announce.go`, `announce_test.go`, `loop.go`, `internal/fabricengine/beamline.go`, `doc.go`, `internal/orchcli/ownerpause.go`.

## Scope guard

Every batch's scope guard listed only the deviations above, informational and none a stop reason.
