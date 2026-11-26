# Review the Describe text inside the Describe row, and catch the rest of what the parent finds at PR-Review earlier

`review-describe-text` · Note · enhancement, landing, burler

Imported from [issue #225](https://github.com/Knatte18/loomyard/issues/225).

## Seen (operator, 2026-11-24)

In the C# repos on one machine, a parent asked to review a PR always finds something.
That points to a design flaw: if the parent finds something, an earlier quality check missed it.

The operator's guess was that the delivered code often does not cover everything the Discussion's decision record asked for.
The operator judges the finding below, that most of it is the PR text, as not very important, and that it is solved by reviewing the Describe text too.

## What the parent finds (analysis, 2026-11-24)

An analysis read 25 PR-Review rounds with findings, across four hubs; one more hub had no PR-Review records.
About 65 findings are itemised in readable text; about 29 more were in scratchpad files that no longer exist.
The task worktrees and the Sketches of the landed tasks are gone, so the only full source is each hub's creel mail journal, and the gate attributions are mostly inferred.
The full report is a local file and is not committed.

- About half of the findings (32 of 65) are about the PR description, not the code:
  internal process or tool names in the text (12), wrong or overreaching claims (12), and content the task asked for that the text omits (8).
- The rest are tests missing or too costly (10), wrong behaviour (8), doc or comment drift (7) and scope the Sketch did not hold (8).
- The operator's guess holds only in part: 4 of 25 rounds cite a Sketch or task requirement that was missed, and all four concern the PR text or scope, not code behaviour.
- A finding is often sent twice because the agent edits the summary file but not the GitHub PR body (about 5 rounds).

## Idea

The Describe row writes the change description, which becomes the PR title, the PR body and the landing commit message.
It is a seat table (`DescribeTable` in `internal/landingshed/describe.go`) whose session already reads the decision record, and its only gate is mechanical (`ValidateDescription`); no agent reviews what the text says.

1. Review the Describe text inside the Describe row with a second agent, an assistant seat, and send its findings back to the writer; Describe is the last step that uses an LLM, so no review runs after it.
   The reviewer checks that the text names no internal process or tool (board, task, scoper, Sketch, decision record, "item N of"), that every claim holds against the diff and the task's report or measurements, and that every deliverable the Sketch says to state in the description is there.
   The mechanical gate stays in front of it.
   This covers the 32 findings about the text.
2. Sync the GitHub PR body from the description on every push, and verify the two match (about 5 re-sent rounds).
3. Run the mutation or redundancy tier on every C# task, and give the Plan a tier0 cost budget (10 test findings).
4. Have Discussion record what the parent will check: measured figures, doc statements and test intent.
5. Let a Code-Review finding the parent already ruled in scope reach a fixer; today a decision finding spawns none.
6. Re-check a constant quoted in a doc when the case it was measured on changes.

The issue on taking the parent out of PR-Review leaves these checks to the cold-start reviewers, so they must run before PR-Review, and a task's scope closes at its first Code-Review round (see the issue on closing a task's scope).
