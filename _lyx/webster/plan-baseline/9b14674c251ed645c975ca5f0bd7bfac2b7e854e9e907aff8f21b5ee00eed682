# Card 18 — landing-hubwide

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configreg#fingerprintOf`
- `internal/configsync#ReconcileHubWideAt`
- `internal/loomcli#loomCLI.wire`
- `internal/configreg/configreg_test.go`
- `internal/configsync/configsync_test.go`
- `internal/loomcli/wiring_test.go`
- `internal/configcli/configcli_integration_test.go`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/landingshed#LoadConfig`
- `internal/fabricengine#BoardDir`

**Intent:**
Make `landing` a hub-wide module, so no session in a pair can weaken `publish_verify`, or set `require_pr_to_base` or `squash`, through its own tree (D10).
`configreg.Module` gains a `SeedsFromPrime` flag marking a hub-wide module whose per-worktree copy was in effect before it moved hub-wide; `fingerprintOf` hashes it with the other flags.
`Modules` marks `landing` `HubWide` and `SeedsFromPrime`.
`ReconcileHubWideAt` seeds an absent hub file from the prime's copy for a module with open maps or with `SeedsFromPrime`, so a hub's first reconcile keeps the prime's `publish_verify` (this repo's `go test -tags tmux ./...`); its doc comment's no-open-maps sentence is restated to match.
A pair's leftover copy is reported retired or divergent by the existing reconcile, and nothing reads it.
`loomCLI.wire` loads landing config from `fabricengine.BoardDir(location.HubPath)`, so Publish and Finalize read the hub's file.
`configreg_test.go`'s hub-wide pin gains `landing` and its fingerprint table a flipped-`SeedsFromPrime` row; `configsync_test.go`'s seed table gains a `landing` row seeded from the prime and one seeded from the template with no prime copy.
`wiring_test.go`'s landing seed writes the board dir's `landing.yaml`, and its landing assertion loads from the board dir; a step pins that a pair `landing.yaml` setting `publish_verify: ""` does not reach the wired config.
`configcli_integration_test.go`'s refusal table gains `lyx config landing --set publish_verify=` in a task pair, refused with the hub file unchanged.
`internal/landingshed/doc.go` states that landing config is hub-wide and read from the board dir.
Bound: per-pair landing overrides are gone for every key; a landing change is an operator or hub-orch `lyx config landing` from the prime, committed in `_board`, effective for every pair at once.

**ImpactSummary:** One registry flag and one seed rule in reconcile; Publish and Finalize read landing config from the board dir, and the registry fingerprint change triggers a hub reconcile at the next start.
