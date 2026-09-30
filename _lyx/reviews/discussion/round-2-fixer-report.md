# Fixer report — Discussion-Review round 2

Every finding in `round-2-review.md` is fixed.
`lyx loom validate-discussion` exits 0 on the result.

## Fixed

- **F1 (BLOCKING):** D8 now dispositions a finding no record-batch saw at run exit.
  A policy finding becomes a recorded warning and the run keeps its outcome.
  A correctness finding demotes Master's `done` to `stuck` with a stuck reason naming each finding and suspect path, and its way forward is a `git` revert on the warp, then a re-step of the Webster row.
  The rationale explains why the integration fork reaches only run exit and why a bare re-step is not enough.
  Two must-cover scenarios are added: an integration-fork `fabric-reference` ending `done` with one run-level warning, and a post-last-batch Master `parent-write` ending `stuck`, then `done` after the revert and re-step.
- **F2 (MEDIUM):** D5's recording path now covers both audit sites: a run-level warning list in `state.json` beside the per-batch ones, the raising verb's envelope (record-batch's or `lyx webster run`'s), and the shared `## Audit warnings` section.
- **F3 (LOW):** Support-log ledger item 18 is resolved: `manifest/roadmap.md` carries no refusal-audit item, so the roadmap does not move.
- **F4 (NIT):** The Scope Out bullet on the step refusal-kind vocabulary is removed, and the Constraints bullet keeps the rule without the "five values" tally.

The support log gains a Round 2 entry, one rejected alternative (a plain run-exit error cleared by a re-step), and ledger rows 22–23.

## Deferred

None.
