# Merge main into the beam adopt-build-resplit

Brings the parent branch `main`, at `9ce7233b`, into the beam `adopt-build-resplit` as a true merge.

## Conflict resolution

The merge had one conflicted path, `internal/machinename/machinename.go`.
The beam had added `idBytes` beside the name-length constant, and `main` had exported that constant as `MaxNameLength` for the cross-machine creel code.
The resolution keeps both: `idBytes` from the beam and the exported `MaxNameLength` from `main`.
Every use of the length in the file already reads `MaxNameLength`.

## Fix after the merge

The beam had removed `machinename.Names` once its last caller moved to `machinename.ReadIdentity`.
`main` brought in two new callers, the creel Slack bridge in `internal/creelcli/slackbridge.go` and the Slack relay wiring in `internal/slackcli/wiring.go`, so the merged tree did not compile.
The follow-up commit restores `Names` unchanged, as a thin wrapper over the existing `resolveNames`, and leaves both callers as `main` wrote them.
