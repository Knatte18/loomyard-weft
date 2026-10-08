# loom resume after circling accept names approve/reject, and circling accept says resume with loom start

`circling-resume-message` · Note · bug, loom

Imported from [issue #446](https://github.com/Knatte18/loomyard/issues/446).

Seen in plan-glyph-precision (batten run, Discussion-Bouncer round 5 escalated, cause budget):

- `lyx loom circling accept plan-glyph-precision` printed `"resume":"lyx loom start"`.
- `lyx loom resume` in the task worktree then refused: "the run is awaiting a decision, which resume does not give; run \"lyx loom approve\" or \"lyx loom reject\" ...", although the decision was already recorded and approve/reject are PR-Gate verbs, not the circling decision.
- The run moved on to Plan-Write only after the driver stepped it.

Expected: after `circling accept`/`continue` under a watching batten run, both outputs name the same next step (the driver's step, or `lyx loom resume`), and `loom resume` either resumes on the recorded circling decision or says that the decision is recorded and the driver will step it.
Role.md tells the hub to run `lyx loom resume` after a circling decision, which this refusal contradicts.
