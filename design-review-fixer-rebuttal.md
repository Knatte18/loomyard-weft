# A burler fixer may rebut a finding, and the next round answers it

`review-fixer-rebuttal` · Note · feature, burler, stencil

## Seen (orch, 2026-11-17)

A burler fixer fixes a finding, leaves it undone or defers it to the parent, but its argument against a finding never reaches the next review round.
The reviewer then sees the same code and can raise the same finding again, which feeds circling.
Source: the survey F17 (`orchestrator-survey-ideas`), from ralphex's external review, where Claude fixes each finding or argues against it and the rebuttal goes into the next round.

## Idea

- The fixer may answer a finding with a rebuttal: why the code stands, with file and line.
- The next round's reviewer gets each rebuttal beside its finding and either withdraws the finding or answers the rebuttal.
- It applies to every reviewer engine, Codex included.
