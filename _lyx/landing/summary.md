# Bouncer review loops escalate to the parent, not a human

A Bouncer review loop no longer halts a run for a human on something the parent orchestrator can settle.
The judge rules `CIRCLING` only from a configured checkpoint round on, and only on gating findings that recur unchanged or reopened.
A rising finding count or new keys alone are never circling.
A `CIRCLING` ruling and a review budget spent without convergence both return `awaiting` with an escalation brief, which a one-shot fork of the parent settles.
The `loomcli` smoke tier is green again.

Closes #359

## What changes

- **Checkpoint judging.**
  Below the checkpoint the judge stencil offers only `CONVERGED` or `CONTINUE`; from the checkpoint on it also offers `CIRCLING`.
  Go renders the decision rule per round, so the stencil stays conditional-free.
  The facts file lists, per recurring open key, the rounds it was open in.
  A Go guard reads a `CIRCLING` verdict as `CONTINUE`, with a Warn, when the round is below the checkpoint or the judge's ledger has no gating finding on an open key spanning the judged round and an earlier one.
  The guard only narrows; it never produces `CIRCLING` or `CONVERGED`.
- **Config.**
  `loom.yaml` gains `review_circling_checkpoint` (default 3) and `review_max_bounces` (default 5), both positive integers.
  The recipe's review-segment rows drop their literal `max_bounces`, and the recipe assembly applies the config budget to Discussion-Review, Plan-Review and Webster-Review.
  PR-Gate keeps its recipe budget.
- **Budget escalation.**
  When the judge rules `CONTINUE` and the `Bounces` seam reports the budget reached, the Bouncer escalates with cause `budget` instead of the `Stuck` that Shed turns into a `bounce-budget-exhausted` block.
  The seam's budget is the config-applied one, so it agrees with Shed.
  An unwired seam, an `ok` false read or an errored read still falls to Shed's generic block.
- **Escalation artifacts.**
  The Bouncer writes `round-<N>-escalation.md` from a new stencil and returns `awaiting` with a one-line `parent_notice`, rendered from a second stencil.
  Shed persists `parent_notice` on the status file and step envelope, set only on `awaiting`.
  The notice interpolates only the slug and brief path.
  `ly-drive` sends the notice to `LYX_PARENT` when both are present; otherwise the stop report and Reason reach the operator as before.
- **Settling.**
  `lyx loom circling accept|continue` settles an escalation of either cause and records the cause in the decision file; legacy decision files read as `circling`.
  A `continue` on a budget escalation grants exactly one more round (both `Stuck`s are `BudgetExempt`); the next judge call escalates again.
  The fork resumes the run with `lyx loom start`.
- **`lyx loom decision add`.**
  A new verb appends one entry, headed `Added after Discussion (<by>, <date>): <title>`, to the decision record's `## Decisions` section through `discussionparser.AppendDecision`.
  It re-runs the `validate-discussion` check, restores the file on a finding, and commits through the discussion commit seam.
  The Plan-Write stencil names it for scope added after Discussion.
- **Selfreport.**
  The `recurring-finding` anomaly, its detector input and its dedup branch are removed, so a recurring ledger finding no longer files a GitHub issue.
- **Smoke tier.**
  The smoke tests counted strands by exact name; they now match through `agentname.Matches`, as production does.

Docs follow in `internal/shedadapters/doc.go`, `internal/shedengine/doc.go`, `internal/loomrecipe/doc.go`, `plugins/ly/skills/ly-drive/SKILL.md`, and the `refusal-spec.md` and `loom-status-spec.md` contracts.

## Design decisions to know

- A `CIRCLING` verdict at a spent budget escalates with cause `budget`, not `circling`, so the `continue` on it is the exempt one.
- `accept` ends the segment unconverged at that round, including on a budget escalation whose verdict was plain `CONTINUE`.
  The decision file records the cause.
- `decision add` is append-only, and `--by` is an unverified label.
  No reviewer reads marked entries differently; only the heading and the committed diff expose them.
- `internal/treadleengine`'s own `CIRCLING` judge and the PR-Gate/PR-Rework segment are untouched.

## To check

- The new `loom.yaml` keys land in this task's weft only; the parent's `loom.yaml` may need `lyx config reconcile` before strict load defaults them.
- The D2 guard trusts the judge's own ledger keys; a judge that renames a key each round reaches the budget escalation instead.
- A fork that records `continue` repeatedly extends a loop one round per decision with no cap; only the decision files show it.
- `crucible/review-prompt-template.md` still lists "a recurring finding" among the anomalies that file a GitHub issue.
- The first verify run failed in `internal/reedengine.TestNaming_TaskWorktreeStrandNameTitleAndEnv`: the installed tmux 3.4 rejects `allow-set-title`.
  The commit `2b09f7e1` cherry-picked from `main` makes that test tolerate it; confirm the gate is green with it.
- `ly-drive` relaying `parent_notice` to a live `LYX_PARENT` and the fork's `lyx loom start` resume were not exercised end to end.
