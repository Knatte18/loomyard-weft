# Push retries, never halts

All eleven batches of the plan recorded `done`, ending at `9da38458520189249aea853e1912bd70e26a14f6`.
No batch needed a re-fork or a recovery.
No digest listed a departure or a rename.

The batches covered the push timing config and the driver's network wait, the waiting-run readers and the owed ledger,
the background pusher's interval and targets, owed code commits and the records hold,
the parent target's landing push, local landing in Finalize, the pull-request owed check,
the prime pull and catch-up changes, the teardown hand-off to the dobby,
and the dobby's push summary with the sync-status field, directives and pattern backgrounds.

## Scope guard

The guard found the files below changed outside their batch's targets.
The findings are informational.

- `01-push-timing-config`: `internal/battencli/run_test.go`, `internal/configreg/configreg_test.go`,
  `internal/fabricengine/config_test.go`, `internal/sheddriver/drive.go`, `internal/sheddriver/driverules.go`,
  `internal/sheddriver/seam_enforcement_test.go`, `internal/shedrun/paths.go`.
- `03-driver-wait-readers`: `internal/battencli/dobbyreads.go`, `internal/loomcli/halt_test.go`,
  `internal/pushledger/ledger.go`, `internal/sheddriver/drive.go`.
- `05-pusher-interval`: `cmd/lym/testonlycode_linux_test.go`, `internal/fabricengine/pushclassified.go`,
  `internal/loomcli/pusher.go`, `internal/runpusher/run.go`, `internal/runpusher/run_test.go`.
- `06-pusher-target-kinds`: `cmd/lym/main.go`, `cmd/lym/main_test.go`, `cmd/lym/testonlycode_linux_test.go`,
  `internal/beamcli/dobbydeps.go`, `internal/beamcli/dobbydeps_integration_test.go`,
  `internal/boardengine/spawn.go`, `internal/boardengine/sync.go`, `internal/fabricengine/commit.go`,
  `internal/fabricengine/export_test.go`, `internal/fabricengine/pushanchored.go`,
  `internal/fabricengine/rejectedtakeover_integration_test.go`, `internal/fabricengine/spawn.go`,
  `internal/fabricengine/spawn_status_integration_test.go`, `internal/fabriclayout/refnames.go`,
  `internal/loomcli/pusher.go`, `internal/loomcli/pusher_integration_test.go`, `internal/runpusher/start.go`,
  `internal/runpusher/start_test.go`, `internal/runpusher/target.go`, `internal/runpusher/targetreads.go`.
- `08-owed-code-commits`: `cmd/lym/testonlycode_linux_test.go`, `internal/beamcli/pr.go`,
  `internal/fabricengine/archive.go`, `internal/fabricengine/cleanup.go`, `internal/fabricengine/coalesce.go`,
  `internal/fabricengine/export_test.go`, `internal/fabricengine/fabric.go`,
  `internal/fabricengine/pushanchored.go`, `internal/fabricengine/pushclassified.go`,
  `internal/fabricengine/pusherreads.go`, `internal/fabricengine/pusherreads_integration_test.go`,
  `internal/fabricengine/recordsgit.go`, `internal/fabricengine/remove.go`, `internal/hubforge/hub.go`,
  `internal/pairteardown/handover.go`, `internal/pairteardown/handover_integration_test.go`.
- `10-parent-push-git`: `cmd/lym/testonlycode_linux_test.go`, `internal/beamcli/dobbydeps.go`,
  `internal/fabricengine/fabric.go`, `internal/fabricengine/landing.go`,
  `internal/fabricengine/landing_integration_test.go`, `internal/fabricengine/pushclassified.go`,
  `internal/loomcli/parentpush_integration_test.go`, `internal/pairteardown/handover.go`,
  `internal/runpusher/parent.go`, `internal/runpusher/parentseams.go`, `internal/runpusher/run.go`,
  `internal/runpusher/targetreads.go`.
- `12-finalize-local-landing`: `cmd/lym/gitrepoboundary_test.go`, `cmd/lym/testonlycode_linux_test.go`,
  `internal/gitrepo/merge.go`, `internal/landingshed/finalize.go`, `internal/landingshed/finalize_gate_test.go`,
  `internal/landingshed/finalize_integration_test.go`, `internal/landingshed/finalize_pr_test.go`,
  `internal/landingshed/finalize_recheck_test.go`, `internal/landingshed/finalize_test.go`,
  `internal/landingshed/finalize_unlocked_test.go`, `internal/landingshed/finalize_verify_test.go`,
  `internal/loomcli/landingdeps.go`, `internal/loomcli/landlocally_integration_test.go`.
- `13-finalize-pr-owed-check`: `internal/fabricengine/destroy.go`, `internal/fabricengine/leftovers.go`,
  `internal/landingshed/finalize.go`, `internal/shedrun/paths.go`.
- `16-prime-pull-owed-tips`: `internal/battencli/catchuppush_integration_test.go`, `internal/battenshed/land.go`,
  `internal/fabricengine/pull.go`, `internal/fabricengine/pushanchored.go`,
  `internal/fabricengine/pushanchored_integration_test.go`, `internal/fabricengine/pushclassified.go`.
- `19-teardown-hands-to-dobby`: `internal/battencli/cancel.go`, `internal/battencli/cancelcmd.go`,
  `internal/battencli/records_integration_test.go`, `internal/battencli/teardownpush_test.go`,
  `internal/battencli/wire.go`, `internal/battenshed/deps.go`, `internal/battenshed/teardown.go`,
  `internal/battenshed/teardown_test.go`, `internal/beamcli/dobbydeps.go`,
  `internal/fabriccli/removelanding_integration_test.go`, `internal/pairteardown/landingstands.go`,
  `internal/pairteardown/teardown.go`, `internal/pairteardown/teardown_test.go`.
- `20-dobby-push-summary`: `cmd/lym/testonlycode_linux_test.go`, `internal/beamcli/dobbydeps.go`,
  `internal/dobbyengine/loop.go`, `internal/dobbyengine/pushsummary.go`,
  `internal/dobbyengine/pushsummary_test.go`, `internal/dobbyengine/seam_enforcement_test.go`,
  `internal/fabriccli/argsarity_test.go`, `internal/fabriccli/roleguard.go`,
  `internal/fabriccli/roleguard_test.go`, `internal/fabriccli/syncstatus.go`,
  `internal/pushledger/syncstatus_test.go`, `internal/reedengine/attach_test.go`,
  `internal/reedengine/bar.go`, `internal/reedengine/windowsize_test.go`.
