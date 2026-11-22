```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the round-2 fix added to the decision record can be carried out from the state it names and leads to progress.

**1. MEDIUM: the orch cannot tell a partial finish from a refusal by reading the cancel's output.**
- Changed text: D4 step 3. A partial finish is "an error whose way forward is to fix the cause and run the cancel again, raised after the run's processes ended". A refusal is "every error the cancel raises before it ends anything".
- Problem: the same wording appears before and after the end, so the orch cannot sort the errors into the two cases.
- Evidence:
  - `internal/battencli/cancel.go:612`, `:620` and `:468` read "way forward: fix the cause it names, then run … again". They come from the origin reconcile and the board pull, which run through `judgeOrigin` at `:314`, before anything is ended.
  - `cancel.go:534`, `:550` and `:555` have the same wording and are raised after the end.
  - `cancel.go:248` says "the cancel ended the run's processes and stopped; way forward: run … again once origin answers". That is a partial finish, but its wording is not "fix the cause", so it fits neither case.
  - `cancel.go:471` and `:759` ("run … again once origin answers", raised before the end) are not in the refusal list.
- Result: an orch going by the output files the pre-end errors as partial finishes and re-runs them. The origin-unreachable partial finish gets no branch at all.

**2. MEDIUM: the stop after the ten-minute bound loses the note's durable copy and gives the operator nothing to act on.**
- Changed text: D4 step 3. "When the records branch is still listed after that, it stops and tells the operator, the entry left a draft under its old parent."
- Problem 1: the stop comes before step 4, so the move note exists only in the orch's `.scratch/`. D2 calls the board copy the durable one, and nothing says the note is kept or handed over.
- Problem 2: no way forward is named for the operator.
- Problem 3: the stop is stricter than the board's refusal. The board refuses only while the branch reads live, and a branch whose tip an `archive/<slug>/*` tag covers reads not live (`internal/fabricengine/remoteleftover.go:81`, `:113-114`).
- Problem 4: for an entry with no stored record, the body upsert never checks the records branch. The stop therefore blocks a write the board would accept.

**3. LOW: the ready-record read assumes the beam's worktree is local.**
- Changed text: D4 step 2. "a beam qualifies only when its worktree's `_lym/landing/beam-ready.json` is absent."
- Problem: a beam owned by another machine has no worktree on the moving orch's machine. The step does not say where the record is read in that case.
- Evidence: `internal/beamcli/cli.go:918-920` shows the record is committed on the beam branch, so it could be read from there instead.

**4. LOW: the hub-pause stop names no step after the operator lifts the hub pause.**
- Changed text: D4 step 3 and D7, "on the hub-pause refusal it stops with its slug pause in place and tells the operator."
- Problem: once the operator lifts the hub pause, nothing says the orch runs the cancel again. The run stays held by the slug pause with no resumption path.

Tool calls:
- grep for "way forward" in `internal/battencli/cancel.go`: my context held only `:290-314` and `:700-724`, not the other error texts.
