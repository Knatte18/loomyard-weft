# Card 5 — cleanup-remote-cli-with-open-prs

**Edit:**
- `internal/fabriccli#runCleanupWithFlags`
- `internal/fabriccli#Command`

**Create:**
- `internal/fabriccli/openprheads.go`
- `internal/fabriccli/openprheads_test.go`
- `internal/fabriccli/cleanupremotewarp_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#Topology.CleanupRemoteWarp`
- `plan:internal/fabricengine#RemoteWarpCleanupResult`
- `internal/githubclient#New`
- `internal/githubclient#ParseOwnerRepo`
- `internal/landingshed#FindPullRequest`

**Intent:**
Implements the CLI half of D9.
With `--remote`, `runCleanupWithFlags` runs `Topology.Cleanup` as today and then sweeps leftover task branches on origin through `Topology.CleanupRemoteWarp`.
It combines both mutation records into the one envelope record.

`openprheads.go` holds the open-PR lookup.
It parses owner and repo from the warp repo's origin URL with `githubclient.ParseOwnerRepo`, builds a client with `githubclient.New`, and lists every open pull request (paginated, bounded by a timeout constant), collecting each head ref whose head repo is the same repository.
It follows `landingshed.FindPullRequest`'s client-and-timeout shape without importing `landingshed`.
The lookup sits behind a package-level function variable so tests can substitute it, the same seam style `fabriccli` uses for its spawn helpers.
Any failure (non-GitHub origin, no token, network error, timeout) returns an error.
`runCleanupWithFlags` then skips `CleanupRemoteWarp` entirely and sets `warp_skipped_reason` naming the cause, while the weft-branch sweep still runs and the verb's exit code is unaffected.

Envelope: `warp_entries` (always an array; each entry's `branch`, `candidate`, `deleted`, `reason`, `error`) and `warp_skipped_reason`.
A non-empty entry `error` under `--apply` makes the verb exit non-zero, alongside the existing local and remote failure synthesis.
The `cleanup` help text documents the task-branch sweep, its candidate rule and reasons, the GitHub-unreachable skip, and that a `--remote` dry run now makes read-only network calls (`ls-remote`, the PR listing).

Bound: the new path can delete only what `CleanupRemoteWarp` classifies as a candidate.
An unreachable GitHub fails toward keeping every task branch, and `--force` still answers no gate.

Tests:
- `openprheads_test.go` (untagged) covers the head filtering over a stubbed PR list: a fork's head is excluded and a same-repo head is included.
- `cleanupremotewarp_integration_test.go` covers `cleanup --remote` with the lookup seam failing: every task-branch deletion is skipped with `warp_skipped_reason`, and weft orphans are still swept.
  With the seam returning a set naming one branch, that branch is reported with the open-PR reason, and `--apply` deletes only the landed candidate.

**ImpactSummary:** cleanup --remote gains a second sweep and two envelope keys; cleanup without --remote is byte-for-byte unchanged.

**Commit:** `5: fabric cleanup --remote sweeps landed task branches on origin`
