# Alignment agent settles parent review and PR-Gate without the orch

The hub orch is the largest token spender, and most of its wakes are parent reviews and PR-Gate reviews that need no judgment.
A cold-start alignment agent, spawned by Go at those two points, now checks the run against the board, the records of the other runs in flight and what landed on the parent since the run's base.
It approves, or rejects on a scope deviation, when the run overlaps nothing in flight.
It escalates to the run's parent, today `ly:orch`, only on a substantive overlap or a named question it cannot settle.

## What changes

- **Parent review.** The `parent-review` gate spawns the agent instead of typing the delivery prompt.
  `approve` and `reject` are recorded by Go; `reject` re-prompts the writer under the existing `attempts: 2` cap.
  `escalate`, a timeout, a dead agent or an invalid verdict falls back to today's delivery prompt, and the brief names the alignment report or says why no verdict came.
  A verdict the parent records while the agent runs wins and the agent is stopped.
- **PR-Gate.** A new `PR-Align` row between `Publish` and `PR-Gate` in `loom-recipe.yaml`; darn's recipe is unchanged.
  An approve writes the approval record through the write path shared with `lyx loom approve`, so `PR-Gate` lands the run with no orch turn.
  A reject writes the rejection record and routes to `PR-Rework`, once per run; a second agent reject is applied as an escalation.
  An escalate, timeout, moved head or pull request, or changed decision snapshot writes no record, and `PR-Gate`'s awaiting notice names the alignment report.
  An approve whose report names superseded board entries queues one orch notice naming them.
- **`lyx loom peers`.** A read-only verb that lists the other in-flight runs (changed paths, which records exist) and the parent's commits since the run's base.
  `lyx loom peers show <slug> --record decision|plan` prints one peer record, so an agent never reaches into another worktree.
- **`loom.yaml` keys.**
  `alignment` (default `opus[high]`), `alignment_timeout_min` (default 20), `skip_alignment` (default false) and `alignment_pr_approve` (default true).
  `skip_alignment: true`, or no board dir, sends both points to the orch exactly as before.
  `alignment_pr_approve: false` withdraws the PR-Gate landing and makes an agent approve an escalation.
  The coherence check refuses `alignment_timeout_min` at or above `parent_review_wait_min`, `run_settings` overrides included.
- **Orch stencil and `CLAUDE.md`.** Parent review and PR-Gate reach the orch only as escalations carrying the alignment report.
  The orch records an operator decision in the same turn: in the board entry's body when no run holds it, with `lyx loom decision add <slug> --by parent` when a run holds it, as a board note naming the task when that verb refuses for lack of a record.
  The `CLAUDE.md` production paragraph now names the agent's PR-Gate approve as a second standing route to `prod`.

## Design decisions to review

- **Go applies the verdict, not the agent.** The agent writes only a verdict file and a report, at per-spawn token paths.
  Go reads a verdict only from the attempt it spawned and only after the spawn, and at PR-Gate only for the head and pull request recorded at spawn.
  An approve is applied only against an unchanged decision snapshot (decision record plus board notes naming the task); otherwise it becomes an escalation.
  A deliberate forgery by an agent in the same pair is not defended against; that is no wider than today, since the approval verbs carry no caller guard.
- **An agent approval can reach `prod`.** At PR-Gate, batten's Land row then runs the hub's `deploy`.
  The bound is the same check the orch's PR-Gate fork ran, the once-per-run reject, and `alignment_pr_approve: false`.
- **Config layering.** `alignment`, `alignment_pr_approve` and `skip_alignment` are closed to the local layer and `run_settings`, because a weaker model or a re-opened auto-landing hollows the approval gate.
  `alignment_timeout_min` is open, since a shorter timeout only routes a review to the orch.
- **`lyx loom peers` has no audience annotation**, because `clihelp.Invocable` treats a runnable group with no `Use` token or flag as non-invocable; `peers show` carries `AudienceRole`.
- `internal/parentreview` keeps its import set: the gate reaches the spawn through a closure wired in `internal/loomcli`.
- `internal/loomcli/parentreview_integration_test.go` sets `env.ParentReview.Aligner = nil`, since the wired aligner would otherwise spawn a real session there.
- Rows for `internal/landingshed` and `internal/loomcli` in `cmd/lyx/testdata/test-budget.yaml` were raised.

## Known risks

- A peer that has not reached Discussion-Write has only its board entry to compare, so an overlap there can be missed.
- Spend per review rises while orch wakes fall; no measurement exists yet.
- An escalation late in the `parent_review_wait_min` window leaves the orch less time before the gate passes with a Warn.
- `runs-overview` also edits `internal/battenshed`'s notice code and the `battenshed` and `loomcli` doc files; expect a merge there.
- `orch-split` will restructure the orch stencil edited here.

## To check by hand

Nothing below was run by hand; each is a finding a review left open or a point the automated tiers do not cover.

- `closeAlignment` in `internal/parentreview` when the gate has no `Aligner`: the record is settled by the fix in `6fe9aec37`, but a record left `running` by a nil aligner should be confirmed gone.
- The note recorded when the head moved (`internal/landingshed/alignmentnote.go`) names the moved head, not the spawn head.
- A poll that errors on closing the attempt, and a failed snapshot read, produce no recorded outcome; check that the fallback still reaches the orch.
- The `PR-Align` call branches and the session start/stop guards have thin test coverage.
- A decision recorded while the agent runs: the board's run lock against a board-note decision, and the window during `PR-Align` before the snapshot is compared, are design points left open; confirm a decision recorded in either turns the approve into an escalation.
- An end-to-end run with a live agent through tmux: the integration fixture plays the parent by hand, so a real spawn, verdict and landing has not been exercised.
