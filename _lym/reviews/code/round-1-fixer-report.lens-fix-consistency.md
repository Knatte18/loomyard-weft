Task: check the fix set for agreement with its surroundings (lens fix-consistency).

no findings

- `internal/adoptcli/doc.go:63` still matches `readDeployState`: an empty `deploy`, or `deploy_branches` not listing the prime's branch, reads nothing further.
- The only callers of `readDeployState` are `deps.go:162`, which passes `runningLymBuild`, and `export_test.go:19`, which passes the test's reader. Both match the new signature.
- The `deployFailure` messages are the same strings `runAdoptDeploy` produced before, so `doc.go:66` and the failed-deploy test row's `wantErr` still hold.
- The `docs/overview.md` sentence and the `PATTERN-cli-cobra.md:36` list agree with `deploy.go`'s imports and `doc.go`'s step order.

Tool calls:
1. One grep over the repo for `readDeployState`, `runningLymBuild` and `deployFailure`, plus the deploy lines of `doc.go` and `git show --stat` of the four commits. My context lacked the callers outside `adoptcli/deploy*`, `doc.go`, and the commits' file lists.
