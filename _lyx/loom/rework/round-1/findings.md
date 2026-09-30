# Review of PR #321 (refusal-audit): fix needed

Build, vet and the full integration suite pass; the defects below are behaviour the tests do not cover.
Fix every numbered item, with a test that reaches each fix.

## 1. HIGH — `goto` can skip review and approval, and can reach teardown

`internal/shedengine/goto.go:58-68` accepts any target from any state except done, including awaiting at PR-Gate and running.

- Awaiting at PR-Gate, `lyx loom goto --to Finalize` then a step merges the task branch unreviewed: Finalize (`internal/landingshed/finalize.go`) never reads the approval record, only PR-Gate does.
  `--to Publish` or `--to Describe` skips Webster-Review the same way.
- `lyx batten goto <slug> --to Worktree-Teardown` (help example, `internal/battencli/cli.go:161,187`) makes teardown reachable from a failed or blocked Run-Shed, breaking the batten recipe header's rule that teardown is unreachable from any failure path.
- A step-driven run sits at running with the run lock free between steps, so goto can race a live ly-drive driver; the help says "halted run" but nothing checks it.

Fix: refuse awaiting and running; allow only targets at or before the current main-line row (never past a review or approval row: Discussion-Review, Plan-Review, Webster-Review, PR-Gate); refuse Worktree-Teardown unless the child run is done, and drop that help example.

## 2. HIGH — fabric-reference audit findings only warn, yet can rewrite run state

`internal/websterengine/audit.go:275`: `ClassifyViolation`'s default classes a fabric-reference finding as policy.
A fork or Master running `git -C <fabric sibling> checkout HEAD~1 -- webster/state.json`, or `rm` on a report, now warns and records once; re-running card verifies cannot detect a state rewrite.
The same write through Write/Edit resolves through the `_lyx` link and is correctness, so the two routes disagree.

Fix: class mutating fabric-reference commands as correctness (git checkout, reset, rm, commit, merge, stash; rm, mv, or a redirect into that path; mutating `lyx fabric` verbs); keep read-only commands as policy.

## 3. MED — `rebaseline` compares card IDs only

`internal/websterengine/rebaseline.go:77` compares `NN-slug` strings in `BatchState.Cards`, so rewriting the body (targets, Verify) of an executed or in-flight card while keeping its slug is accepted, contradicting `refusal-spec.md` ("cards removed or changed") and the verb's help.
A weakened Verify then gives a false pass in `rerunCardVerifies` and the done-checks.

Fix: store a content hash per card in `BatchState.Cards` at begin-batch and recover-batch, and compare hashes.

## 4. MED — way-forward texts that do not work where they are reached

- a. `internal/loomcli/approve.go:89`: "push or sync the task branch (lyx fabric push)" is wrong; that verb pushes only the `_lyx` side, while the PR head is the code task branch. Name a plain `git push` of the task branch, or a pull when the remote is ahead.
- b. `approve.go:77,80`: "lyx loom step opens it at Publish" loops: the run sits at PR-Gate, which returns Awaiting when there is no PR or it is closed (`internal/landingshed/prgate.go:81-83`), so it never reaches Publish. Name `lyx loom goto --to Publish`, then step (a backward goto, legal under item 1).
- c. `internal/shedengine/run.go:163,300`: the budget-exhausted and missing-producer texts name `lyx shed goto --to X` without the run-id, which addresses `self` and misses a slug-addressed (batten) run. Include the run-id.
- d. `contracts/specs/refusal-spec.md:113` says "at Publish"; the code says PR-Gate.
- e. `missingStatusWayForward`: for batten, `lyx shed seed` does not create a status file (batten's own run/step does), so the named way forward loops. Name batten's own verb.

## 5. MED-LOW — a run-exit correctness halt clears on a bare re-step

`internal/websterengine/runlevel.go:982` deliberately does not record the finding, and finding identity is keyed on the Master session, so `lyx webster run` without reverting the named paths starts a new Master that re-audits only its own transcript and reports done.

Fix: persist these findings as pending and re-check the suspect paths (or require an explicit acknowledgement) at the next run exit.

## 6. LOW — stale Master stencil

`contracts/stencils/webster/webster-template-master.md:151` still lists "a symbol this batch deleted that the remaining plan still references" as a `batch_failed` done-check; `postBatchChecks` now records it as a later-card-drift warning.

## 7. Operator rule — "warp" and "weft" belong only inside fabric

Nothing outside fabric may use these words, in code, tests, specs, docs or skills.
Reword every occurrence this branch adds outside fabric:

- `contracts/specs/refusal-spec.md:12`, `:35`, `:56` (and any other line in that file)
- `internal/preflightshed/wayforward_test.go:2` (comment) and `:22` (fixture reason `"no weft"`)
- `internal/webstercli/verbs_test.go:1192-1203` (helper names, comments and messages)
- `internal/websterengine/auditseverity_test.go:36,37,40,76` (local variable)

The existing `WEFT_SKIP_GIT` environment variable name may stay as is.

## Nits

- `docs/overview.md:247` still says "run/step/status/pause" for shedverbs; add goto.
