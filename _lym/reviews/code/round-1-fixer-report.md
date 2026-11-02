```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6]
disputed: []
deferred: []
```

## Changes

Commits, newest last: `fdfc3e5d` (F2, F5), `121a209f` (F1, F6), `1c30bd8d` (F3), `bf8c0217` (F4), and one follow-up for the self-check.
F2 and F5 share `deploy.go`, and F1 and F6 share `adopt_test.go`, so each pair is one commit.
`lym gate test ./internal/adoptcli --tier 0,1` passes.

- **F1** (fixed differently, second row).
  The held-root row `AStopHoldingRootBackDeploysNothingAndWritesNoOwner` is added to `TestAdopt`: a stopped root task holds root back, the build is stale, and the row asserts no deploy and no owner write.
  The proposed second row, a prime already owned here with a stale build, is not added.
  `newWorld` sets `Prime: here`, so the existing deploy row already runs in that case.
  That row is renamed `RootDeploysALymOlderThanThePrimesHeadBeforeTheOwnerWriteWhenThisMachineAlreadyOwnsThePrime`, and a second row would duplicate it.
- **F2** (fixed differently, seam).
  `TestReadDeployState` is added in the new `deploy_integration_test.go`, a tier 1 hubforge test, because the landing config's deploy keys are barred from the worktree layer and are seeded into the Bolt checkout.
  Rows: no deploy command, a branch list without the prime's branch, a build of an ancestor, of the head, of a commit the prime lacks, and a build read error.
  The proposed executable-path seam is not used.
  `readDeployState` takes a `readInstalled func() (buildvcs.Identity, error)`, because a `go test` binary carries no VCS stamp and no stamped binary can be built for a row.
  Production passes `runningLymBuild`, which does `os.Executable` and `buildvcs.OfFile`.
  `export_test.go` exposes `ReadDeployState` and `DeployState`.
- **F3.** `battenshed` and `buildvcs` are added to the `adoptcli` line of `pattern/PATTERN-cli-cobra.md`.
  The self-check led to a second edit, below.
- **F4.** `docs/overview.md` adopt bullet gains the deploy step between the holds and the owner write.
- **F5.** The result-to-error mapping moved into `deployFailure(result, err)`, and `TestDeployFailure` in `deploy_test.go` covers a start error, a timeout, a non-zero exit code and a zero exit code.
  The `testing.Testing()` guard stays in `runAdoptDeploy`.
- **F6.** The failed-deploy row seeds a reconcile mutation and asserts `got.Partial` and that `got.Mutations` equals it.

## Disputed

None.

## Deferred

None.

## Self-check

Four forks returned reports, saved as `round-1-fixer-report.lens-<lens>.md`.

- fix-proposal, LOW: the `docs/overview.md` sentence said "With root", but the gate is `rootGoing` and needs a `deploy` command listing the prime's branch. Fixed: the sentence now states those conditions.
- fix-proposal, LOW: the `PATTERN-cli-cobra.md` line says "takeover functions of", which `buildvcs` and `battenshed` are not. Fixed differently: the review said to keep the wording, but "takeover" is dropped so the line is true of every package on it.
- fix-proposal, outside the lens: `runningLymBuild` is reached by no `adoptcli` test. Not a finding under the rubric: a two-call wrapper whose `OfFile` has its own test in `buildvcs`. Nothing changed.
- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
