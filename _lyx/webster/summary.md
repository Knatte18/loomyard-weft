# hub-workspaces: one VS Code workspace file per hub, portals retired

All four batches reached `done`; no recovery or re-fork ran, and no digest carried departures or warnings.

- **01-hub-workspace-file** (head `acf5697d3`): one fabric function writes `<hub>/<RepoName>.code-workspace` under a hub lock, and the prime spawn uses it.
  Deviations: `internal/fabricengine/hubworkspace.go`, `internal/fabricengine/hubworkspace_integration_test.go`, `internal/fabricengine/hubworkspace_test.go`, `internal/ideengine/spawn_hub_integration_test.go`.
- **02-verbs-regenerate** (head `3467b98da`, cards 2 and 3): the topology verbs and Reconcile regenerate the file, warning on failure.
  Reconcile restores launchers only, skips local pairs and reports `launchers_restored`.
  Deviations: `internal/fabriccli/adoptremote_integration_test.go`, `internal/fabricengine/hubworkspace_integration_test.go`, `internal/fabricengine/launchers.go`, `internal/fabricengine/livestate_verbs_test.go`, `internal/fabricengine/reconcile_stale_registration_test.go`.
- **04-retire-portals** (head `aa29a4859`): no verb creates or removes a portal link; the portal helpers, accessor and their tests are gone.
  Deviations: `internal/fabricengine/add_rollback_adopt_test.go`, `internal/fabricengine/verbs_regenerate_integration_test.go`.
- **05-retired-artifact-sweep** (head `c7e16fbbe`): Reconcile sweeps retired portal links and the old workspace file, reports `hub_details`, and regenerations warn while they remain.
  Deviations: `internal/fabricengine/portals.go`, `internal/fabricengine/portals_sweep_integration_test.go`.

The scope guard's only findings are the deviation lists above, all informational.
