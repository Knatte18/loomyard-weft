# Fixer report — plan review round 1

Every finding was fixed in the plan files under `_lyx/plan/`.
`approved:` in `00-overview.md` was left untouched.

## Changes

- F1 (`00-overview.md`): the `## verify:` tmux line now also runs `./internal/loomcli/...` and `./internal/webstercli/...`, the two targeted packages with tmux-tagged tests.
- F2 (`03-reed-unless-name-dormant.md`): `internal/reedengine/doc.go` and `docs/overview.md` joined the targets, and the Intent and ImpactSummary say both sentences on the old live-only rule are rewritten to the any-strand-in-state rule.
- F3 (`10-verify-wait-note.md`, `12-loom-status-shuttle-wait.md`): card 10 now edits `contracts/specs/loom-status-spec.md` ("The `waiting` key") and `docs/overview.md`, with the Intent stating the three waits, their read order and the unchanged absent cases; card 12 says its shuttle step is the second of those waits.
- F4 (`09-verify-timeout-findings.md`): `internal/websterengine/doc.go` and `internal/landingshed/doc.go` joined the targets, with the Intent stating the timeout outcome each records (webster's gate fails at once with no rerun, landing halts Stuck with the log path).
- F5 (`05-reed-resume-done-when.md`): a new step in the tmux-tier `TestStrandOps_RealTmux` pins `Resume`'s removal from persisted state, `Dropped` and the relaunch of the partly missing case, and the existing resume envelope check in `smoke_staterecovery_test.go` gains the `dropped` key assertion; both files joined the targets.
- F6 (`05-reed-resume-done-when.md`): the Parent condition, its Bound clause and its test row are gone, so the card implements D5 as written.
- F7 (`09-verify-timeout-findings.md`): the reason for the new untagged test file now says the verify gate's only test is integration-tagged and `NewVerifyGate` passes the production timeout.
- F8 (`12-loom-status-shuttle-wait.md`): the empty-result case is dropped from the card's test list, with a sentence that card 10's rows already pin it.
- F9 (`16-batten-respawn-scoped.md`): the Intent states that a retiring driver strand counts as live in the running-and-unconfirmed arm, so the child is adopted, and the test list gains the retiring case.

## Deferred

Nothing deferred.
