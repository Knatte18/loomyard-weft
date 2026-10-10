# Quick recipe: one producer, then the usual PR; shared Publish, Finalize and teardown fixes

`quick-recipe` · Task · feature, bug, loom, shed, batten, fabric, board

## Seen

Small cosmetic or narrow changes, such as `board-priority-headings`, are too small for a full loom run (Discussion, Plan, Webster, review rounds), so today they go to a hand-briefed agent in the prime instead.
That path skips the pair, the preflight, the verify gate, the PR and PR-Gate, and the orch has to brief, watch and land each one by hand.

## Idea

A second task recipe for quick tasks, run through batten and shed like a loom run but with a strongly simplified loom inside:

- the same outer lifecycle: batten creates the pair, preflight runs, and teardown removes the pair after landing;
- one LLM producer that does the whole change in a single session: reads the board entry, edits, runs the tests and commits;
- the usual Publish: verify gate, PR, the orch's PR-Gate with a reviewer fork, then Finalize.

No Discussion, no Plan, no Webster, no Burler rounds.
## Operator decisions (2026-10-10)

- The operator picks the recipe when the task is created, through the board entry's existing `recipe` field; no size guard.
- A failed verify sends the run back to the same producer to fix it, up to a configured budget of attempts; past the budget the run escalates to the parent.

## Prime agents block runs (2026-10-10)

A hand-briefed agent editing in the prime leaves it dirty, and `lyx batten run` then stops at Worktree-Create ("source worktree has uncommitted changes") until the agent pushes, as `cli-surface` did while the seats N=0 prompt fix (abe2aa50e) was in progress.
The quick recipe runs such fixes in their own pair, so the prime stays clean; the operator wants the orch's prime-agent fixes to move to it over time.

## Folded in: shared Publish, Finalize and teardown fixes

The quick recipe reuses loom's Publish and Finalize and batten's teardown, so these fixes land with it.

### Verify log per attempt (#502, from `verify-log-overwritten`)

Seen (publish-recovery, 2026-10-10): the Publish verify failed once (exit 1, the #497 subtest re-run), and a later verify run overwrote `.lyx/verify/verify.log`, so Friction-Reflect could not read the original failure and reflected on the later, passing log.
Expected: the verify log is kept per attempt (for example `verify-<attempt>.log`, or appended with an attempt header), so a later run never erases the evidence of an earlier failure.
The quick recipe's verify-retry loop needs this, since its producer fixes the failure the log records.

### Finalize ignores the parent weft's batten records (#501, from `finalize-weft-batten-dirty`)

Seen (publish-recovery, 2026-10-10): after `lyx loom approve`, Finalize stopped blocked with "fabricengine: merge preconditions failed: worktree dirty".
The task pair and its weft were clean; the dirty tree was the prime's weft, holding uncommitted batten records (`_lyx/shed/<slug>/status.json`) of sibling runs.
Batten writes these from the prime while runs are in flight and commits them later through weft sync, so any Finalize that runs between a write and its sync commit hits the precondition.
Workaround today: `lyx fabric commit` in the prime, then `lyx loom resume` in the pair.
Expected: Finalize's merge precondition ignores, or commits first, the parent weft's own batten records, so a sibling run's status write never blocks a landing.

### Teardown clears an abandoned run's claim (from `abandoned-run-claim`)

With `board-lock-running`, a board entry whose status has the run form (`<state> · <producer>`) refuses every scope change.
A run that ends without landing (stopped, abandoned, its pair removed) leaves that status behind, so the entry stays locked until someone clears it by hand (seen in the PR-Gate review of PR #482).
Batten's teardown of a run that did not land clears the claim, or sets a non-run status, so the task becomes editable again.

### Friction-Reflect reads the parent weft (from `reflect-reads-parent-weft`)

Seen (publish-recovery, 2026-10-10): Finalize blocked on "worktree dirty" from the prime weft's batten records (#501), and Friction-Reflect dismissed it as a process signal because the task worktree was clean.
Friction-Reflect also reads the parent weft's status (`lyx fabric status` in the parent) when a halt names a dirty worktree, so it attributes the halt to the right tree.
