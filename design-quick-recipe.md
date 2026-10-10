# A quick task recipe: batten and shed around one LLM producer, then the usual PR

`quick-recipe` · Note · feature, loom, shed, batten

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
