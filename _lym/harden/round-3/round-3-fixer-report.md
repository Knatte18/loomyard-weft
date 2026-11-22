---
dispositions: []
---

# Round 3 fixer-report

Nothing fixed.
The review is APPROVED with no finding, so no disposition is owned.
The orch brief (#2350) says to leave the review's Rejected items unfixed this round, and I did.

## Gate finding: live run started no test binary

The gate finding names `harden-live-3.json`, mark red because `live.started` is 0 for a config naming packages.
No test failed and no source defect exists to fix.

Cause, read from the hub config: `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/config/gate.yaml` has no `harden-live` key under `gates:`.
Commit `02cf337824` in the `_bolt` worktree ("drop the hardener gate keys the hardener rebuild retired") removed it.
Running `lym gate test --gate=harden-live ./internal/fabricengine` here falls back to tier0 and ends with "started 0 test binaries in tier0".
Round 2 started 3 binaries, so the key existed then.

The gate finding was re-delivered after the first report.
I re-ran `lym gate test --gate=harden-live ./internal/fabricengine`: the hub's `gate.yaml` still has no `harden-live` key, and the run still starts no binary.
No repair inside this worktree existed, so I flagged it to `ly:orch` (#2351).
`ly:orch` then restored `gates.harden-live` and `gates.harden-burler-fix` as `{tiers: [0, 1], scope: diff}` in the hub default layer (#2352).
I re-ran the gate: tier1 started 1 test binary, 1582 passed, 0 failed.
The gate finding is resolved by that config change; no source change was needed.

## Disputed

None.

## Deferred

None. The gate finding's repair was a hub config change made by `ly:orch`, outside this worktree.

## Rulings

None.
