# Fabric add: replace archived weft leftovers

This session resumed with all five batches already `done` in the progress trail, so it forked nothing for them.
`lyx webster status` agreed: batches 1–5 (leased-remote-delete, gate-archived-weft-delete, add-remote-preflight, cli-leftover-preflight-envelope, add-replace-archived-weft) were all terminal `done`.
Status also listed a terminal `stuck` entry at number -1 for `01-leased-remote-delete`. I did not investigate it, since batch 1 itself is `done`.

The plan has a `## verify:` section, so I spawned the integration fork.
It wrote its report with `status: FAILED`, `head_sha: fd2c3d3ec01d3ae20eec8bda2d62ee65d945919d` and no deviations.
I did not try to localize or fix the failure and did not re-fork.
Webster triages it after the session ends: it reruns the suite against the plan's starting commit and escalates only a regression to `stuck`.


## Integration suite triage

The plan-level `## verify:` suite failed, but webster's triage did not attribute the failure to this run.

Pre-existing (already failing at the plan's starting commit):

- `github.com/Knatte18/loomyard/internal/fabriccli.TestRunCLI_CleanupRemoteFailureExitsNonZero`
