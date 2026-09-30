# Drive report: fabric-readd-weft-push

**Stop condition:** `state: awaiting` at `Publish`.
This is a human gate, not a failure.

> pull request created; awaiting review, then run `lyx loom approve` and `lyx loom start`: https://github.com/Knatte18/loomyard/pull/312

## Baseline

`lyx shed status` returned `state: blocked`, `current_producer: Webster` and `history_length: 14`.
The block was an integration-verify regression in `cmd/lyx.TestGitrepoBoundary_PinnedRunCallSites` (card `01-leased-remote-delete`).
HEAD already carried the operator fix `fd2c3d3ec cmd/lyx: pin DeleteRemoteBranchLeased in the gitrepo boundary set`, so the first step resumed the run.

## Steps

| # | Producer | Outcome | Next | Trace ID |
|---|---|---|---|---|
| 1 | Webster | done | Webster-Bouncer | df1c772488d796db |
| 2 | Webster-Bouncer | stuck | Webster-Burler | 55b5f069299d324f |
| 3 | Webster-Burler | stuck (round-1 review APPROVED with findings) | Webster-Bouncer | e8fffd23eff7b93d |
| 4 | Webster-Bouncer | done | Describe | fe5662b21b50a71a |
| 5 | Describe | done | Publish | fb2eed1dbb03f19c |
| 6 | Publish | awaiting (PR #312) | Publish | d48a629e6b8e1cfc |

Webster's integration verify still fails on `internal/fabriccli.TestRunCLI_CleanupRemoteFailureExitsNonZero`.
The engine classed it as pre-existing, since it also fails at the plan's starting commit, so it did not block the run.

## Repairs

None.
No error envelopes.
The repairs directory `.lyx/shed/fabric-readd-weft-push/repairs/` was not created.

## Pointers

- friction_dir: `/home/knatte/Code/loomyard-LYXHUB/fabric-readd-weft-push/_lyx/loom/friction` (holds `webster-verify-triage.md`)
- repairs: `/home/knatte/Code/loomyard-LYXHUB/fabric-readd-weft-push/.lyx/shed/fabric-readd-weft-push/repairs/` (empty)
- Webster review: `_lyx/reviews/webster/round-1-review.md`, `_lyx/reviews/webster/round-1-bouncer-ledger.md`
- Landing summary: `_lyx/landing/summary.md`

## Operator choices

1) Review PR #312, then run `lyx loom approve` and `lyx loom start` — this resumes the run into Finalize and Friction-Reflect.
2) Investigate `TestRunCLI_CleanupRemoteFailureExitsNonZero` separately — it fails on the base too, so it falls outside this run's scope.

## Self-report draft (not filed; autonomous mode)

Title: `fabriccli: TestRunCLI_CleanupRemoteFailureExitsNonZero fails on main (integration)`
Label: bug
Body: Webster's integration verify triage for run fabric-readd-weft-push found `github.com/Knatte18/loomyard/internal/fabriccli.TestRunCLI_CleanupRemoteFailureExitsNonZero` failing at the plan's starting commit as well as at the task tip.
It is a pre-existing integration failure that every run touching `./internal/fabriccli/...` will re-triage.

Side note: every `lyx` call logs stencilstore drift warnings (source-ahead for landing-template-describe, loom-template-plan, webster-template-master, webster-template-integration).
The production board copy predates the worktree source until `update-plugins.sh` runs.
