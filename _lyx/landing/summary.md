# loom: route a rejected PR review back into the run

An operator who rejects the open PR with a review file now gets the findings planned as new cards and implemented by Webster through the normal fork/record/audit path.
The run is then re-verified, re-reviewed, re-described, pushed to the same PR, and left awaiting approval again, with no fix made outside loom.

## What changes

- `lyx loom reject <review-file>` is a new verb beside `approve`.
  It refuses unless the run is awaiting or blocked at the gate (or blocked at the rework row), the PR is open at the local HEAD, and the file is non-empty.
  It refuses at the gate once the segment's bounce budget is spent, naming `approve` as the only way forward.
  It writes a rejection record with the findings inline, removes any pending approval record, and never resumes the run; the envelope carries `resume: "lyx loom start"`.
- `Publish` is split.
  `Publish` syncs, verifies, pushes, and creates or refreshes the PR, then returns `Done` once an open PR exists at the pushed head.
  It now edits the PR title and body when the re-run `Describe` output differs, and it no longer reads the approval record.
- A new `PR-Gate` row (`internal/landingshed`) owns every review decision: approval, rejection, and the wait.
  Only a matching rejection makes it `Stuck`, which bounces to the rework row; every other hand-off at the PR is `Awaiting`.
- A new `PR-Rework` row (`internal/loomshed`, stencil `loom-template-rework.md`) turns the findings into cards appended to `_lyx/plan/`.
  It uses the `gate: plan` format check, then an append-only check against the plan at HEAD.
  It commits the cards and a per-round findings directory, re-baselines Webster's plan fingerprint, deletes the rejection record, and routes to `Webster`.
- `approve` now works at the gate row instead of `Publish` and removes a pending rejection record first.
- Batten resumes an awaiting child on a rejection exactly as on an approval, once per decision.

## Design decisions

- `shedengine` is unchanged.
  The gate and rework row share a new `PR-Review` segment with `max_bounces: 5`, and the gate's `Done` happens only on approval or merge, so the budget is five rejection rounds per task.
  `reject` refuses before a sixth `Stuck` can exist.
- Webster is not special-cased.
  `websterengine.RebaselinePlanFingerprint` restamps the fingerprint under the state lock, so Webster's ordinary resume skips completed batches and runs only the new cards' batches, then the integration suite.
- The append-only check reads the plan as committed at HEAD (`fabricengine.CommittedAnchoredFile`, `planparser.CheckAppendOnly`), so leftover edits from a failed attempt are still caught on retry.
  A crash after the commit but before record deletion is recovered on re-entry without appending cards twice.
- Batten's decision identity is the record kind, its timestamp and its head SHA, so an approval and a rejection at the same head in the same second are distinct.
- Rework cards skip `Plan-Review`: the findings are operator-authored, and the resulting diff still passes `Webster-Review`.
- Behavior change for operators: a run halted at the gate re-runs only the gate on `lyx loom start`, so a fix committed by hand is no longer pushed by a plain `loom start`.
  Push it yourself before `approve`, which refuses on a head mismatch, or use `reject`.
  A run already awaiting at `Publish` when this lands re-runs `Publish` via `lyx loom start` before it can be approved.
- Tenter is untouched.

The recipe header comment, `docs/overview.md`, `loom-status-spec.md`, the `ly-drive` skill and CLI help are updated in the same change.

## To check by hand

- Drive a real reject round end to end against a live PR: `reject`, `loom start`, the new cards' commits, the PR title and body refresh, and a second `approve`.
- Coverage of every finding by a card rests on the stencil and on `Webster-Review`; it is not mechanically checked, since findings are free-form.
- Each rejection costs a full `Webster-Review` round over the whole task diff.
- A future grouping batchifier could regroup old and appended cards and invalidate Webster's per-batch state; only the identity batcher exists today.
