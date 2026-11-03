# loomcli: check origin and owners before a run commits or pushes

A loom run that went on after a sleep worked on a stale base, and nothing told it until a push was refused or a landing conflicted.
Every commit and push of a loom run's pair now passes a pre-push check first, where a stale base does harm instead of at the wake.
The covered paths are the per-transition status commit and push, `lym loom commit-records`, and PR-Review's push of the task branch.

The check runs the run-scope origin reconcile from the prime, the one the landing rows already run.
It fetches what origin moved and takes in each of the run's own branches where a fast-forward or replay can.
What follows depends on the verdict on the run's entry:

- **Active:** the push goes.
  One notice line names the run's own branches the reconcile moved.
  A second names the parent branch when origin moved it past this machine's tip, once per origin tip.
- **Owned by another machine:** the run is paused under the owner caller, and the stop names `lym adopt <subject>`.
- **Stopped by a branch:** the run is paused.
  The stop names each of the run's branches, and its parent branch, that stopped with its state, then the way forward and `lym tender resume <slug>`.
- **Ended by origin:** the run is paused naming `lym cleanup --apply`, unless the parent already holds the run's work, as after its own landing.

Every stop posts one notice line to the nearest live orch.
A stopped status transition commits and pushes nothing.
`commit-records` answers an ok envelope with `push_stopped`, with `committed` false when the check stopped the commit and true when it stopped the push after one.
PR-Review halts on the stop as on any push refusal.

## Design decisions

- The check is skipped when pushes or git are off, in the prime, while a pause covers the run, and at Finalize.
  A pause's network gate already holds the push, and Finalize judges origin and its owners itself.
- The push goes unchecked when the check cannot judge: origin unreachable, a failed reconcile, a failed read of whether the check applies, or a failed read of whether the parent holds the run's work.
  The push's own failure is then recorded as before.
- Bound: the check reads origin once per commit and push, so a move landing between the check and the push is caught only by the push's own refusal.
- `fabricengine` exports `BranchState.Stops` and the `RepoCode`, `RepoRecords` and `RepoBolt` constants, which the check reads.
- `tendercli.PausePair` pauses a run with a plain pair entry, and `shedrun.ParentNoticeFile` holds the key of the last parent-branch notice.
- `PATTERN-owner-machine` and its background file now cover a push, and `loomcli`, `tendercli` and `battenshed` docs describe the check.

## To check by hand

Nothing was run by hand, and no real two-machine sequence was exercised.
Run a loom run across a wake where another machine moved origin, and where it adopted the run, and confirm the stop and the notice line.

Findings left open, to check in the diff:

- `code-repo-literal-duplicated`: the code-repository side literal is duplicated; see `internal/fabricengine/failedpush.go` and the `Repo` comparisons in `internal/loomcli/prepush.go`.
- `parent-notice-path-outside-shedrun`: the parent notice record's path must come from `internal/shedrun/paths.go`; confirm no other path is built outside it.
- `pr-review-pushtask-pausepair-untested`: PR-Review's `PushTask` stop and the `PausePair` pause path in `internal/loomcli/landingdeps.go` and `internal/tendercli/deps.go` have thin test coverage.
- `skip-closure-untested`: the `skip` closure in `newPrePushCheck` in `internal/loomcli/prepush.go` is pinned only through `skipsCheck`.
- `stopped-check-callers-unpinned`: the callers that read `errPushStopped` in `internal/loomcli/wiring.go` and `internal/loomcli/commitrecords.go` are not all pinned by a test.
