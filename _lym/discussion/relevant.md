# Relevant files

- `internal/machinename/machinename.go` `Name`, `FromHostname` — the name source this task extends
- `internal/machinename/doc.go` — package doc to update with the machine file
- `internal/machinename/machinename_test.go` `TestSoleSource_NoOtherProductionFileReadsTheHostname`, `TestLeafInvariant_AllowlistOnly` — the scans the change must keep green
- `internal/githubclient/cache.go` `CredentialDir` — today's per-user config directory resolver
- `internal/standalonestate/standalonestate.go` `UserStateDir` — the leaf that gains the config directory resolver
- `internal/boardengine/ownerset.go` `OwnerRequest`, `SetOwners`, `ReadOwners` — the owner write the sweep reuses
- `internal/boardengine/owners.go` `OwnedElsewhere` — owner comparison by string equality
- `internal/hubregistry/hubregistry.go` `Live` — the hubs the sweep visits
- `internal/battencli/dobbyreads.go` `DobbyLandings` — landing-history match by machine name
- `internal/slackengine/keeper.go` `create`, `retireEntry` — why a rename is safe for existing Slack channels
- `internal/adoptcli/doc.go` — the existing owner move, rejected for a rename
- `internal/testkit/tmuxkit/tmuxkit.go` `setTempEnv` — test environment, left unchanged by D8
