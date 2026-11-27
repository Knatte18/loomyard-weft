# Review the Describe text in its row, keep LoomYard terms out, sync the PR body

`describe-review-and-sync` · Task · enhancement, landing, stencil, burler, shuttle · running · Darn-Write

This task builds items 1 and 2 of the Describe-text note's Idea; items 3 to 6 belong to the task that closes scope at Code-Review.
No agent reviews the pull request any more (task `pr-modes-no-agent`), so the Describe text is the last text an LLM checks.

Bundled from the notes below; each part keeps its note's text.

# The Describe text names no LoomYard terms, except the Created using block

Imported from [issue #227](https://github.com/Knatte18/loomyard/issues/227).

## Seen (operator, 2026-11-24)

An analysis of the parent's PR-Review findings in the C# hubs found that about 12 findings were internal process or tool names in the PR text: "item 3 of the task", "the test scoper cannot run C# tests there", "the task's branch point", "The board entry proposed", "The parent recorded...".
The stencil that writes the description (`landing-template-describe`) does not forbid them, so the parent had to tell the Describe session so in each hub.
In one run the parent noted its own earlier instruction had caused it.

The operator's rule: LoomYard, and anything from it, does not belong in the Describe text.
The one exception is the "Created using" block at the bottom.

## Idea

Keep LoomYard's vocabulary out of the description the Describe row writes, and let lym hold it.

- Add the rule to the Describe stencil: the description names no board, task, Sketch, decision record, parent, orch, scoper, gate, tier, perch or lym term, and no "item N of the task".
- Let the Describe row's mechanical gate (`NewDescriptionGate`, which runs `summaryparser.ValidateDescription`) refuse such a term outside the credit block, with the refusal naming the term and the line.
- Leave the credit block as it is: the `credit` keys in `landing.yaml` append it, and the term check skips it.
- Read first how the credit block is appended to the description, and which words the check can match without refusing a repository's own legitimate names.
- The issue on reviewing the Describe text covers claims and missing content; this issue covers the vocabulary.


# Review the Describe text inside the Describe row, and catch the rest of what the parent finds at PR-Review earlier

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


# The GitHub PR body is not updated when the description file changes

Imported from [issue #228](https://github.com/Knatte18/loomyard/issues/228).

## Seen

An analysis of the parent's PR-Review findings in the C# hubs found that the same findings were sent again in about five rounds.
The agent fixed the description file but the pull request's body on GitHub was not updated, so the parent read the old text again.
Seen in five separate rounds across several runs.

## Why it makes no sense

The description file is the one source of the PR title and body.
A PR whose body differs from the file the run holds is wrong from the first push, and a review round is spent on text that has already been fixed.

## Idea

Make the PR body follow the description file on every push, and refuse to end a round while the two differ.

- Read first where `pr update` runs for a task's pull request, and which path writes the description file without calling it (`lym beam describe` runs the `pr update` sequence for a beam; check the task path).
- After each push, compare the GitHub PR body with the file, and update it or halt with a way forward naming the difference.
- Add a test that edits the description file after the PR exists and expects the body to follow on the next push.


# Describe halts failed on an uninformative shuttle run outcome timeout

Imported from [issue #244](https://github.com/Knatte18/loomyard/issues/244).

The Describe step halted the run as `failed` with anomaly `producer-hard-failure` and the message `shedadapters: Describe (seats): shuttle run outcome timeout`.

Why it is abnormal:
- The note carries no detail on which timeout elapsed, how long the shuttle run had been going, or whether the agent was still working.
- A timeout on a seated shuttle run is classed as a hard failure, so the run stopped as `failed` rather than being resumed or escalated.
- The next attempt of the same step completed (`Describe / done`) about 34 minutes later, which suggests the timeout was not a defect in the producer's work.

What to do:
- Include the elapsed time, the timeout limit and the shuttle run's last known state in the halt reason.
- Decide whether a shuttle outcome timeout should be a retryable or `blocked` stop instead of `failed`.

Provenance: task `retire-d9`; note loom-halt.md.


# Describe seat waits out the full timeout after an operator interrupt, then halts as hard failure

Imported from [issue #241](https://github.com/Knatte18/loomyard/issues/241).

The Describe producer (seats engine) halted the run as `failed` with `shuttle run outcome timeout`, after its agent session sat held for about 28 minutes with no output.

Sequence from the trace:

```
22:15:05 shuttle: run started role=describe
22:16:51 shuttle: session state state=idle-stalled cause=interrupt
22:16:53 shuttle: session state state=busy cause=turn
22:16:57 shuttle: turn end held, output files missing
22:16:57 shuttle: held turn end of a turn the operator started; no hold notice
22:16:57 shuttle: session state state=idle-stalled cause=no-output loop=held
22:45:06 shuttle: run finished outcome=timeout cleanedUp=false
22:45:29 shed: step refused kind=producer transient="" error="shedadapters: Describe (seats): shuttle run outcome timeout"
```

The halt note reads `loom halted failed at Describe`, `anomaly: producer-hard-failure`, 12 history entries.

Why this is abnormal:
- The operator interrupted the agent once and typed into the pane, which made the turn "operator-started".
  lym then held the turn end without a hold notice and waited silently until the run timeout.
- The session was `idle-stalled` for the whole wait, and nothing re-prompted the agent for its missing output files or surfaced the state to the operator.
- The timeout was then recorded as a hard failure with `transient=""`, although the cause was an interrupted session and not a producer defect.
- `cleanedUp=false` on the finished run, and the strand was stopped with `live=true` afterwards.

Suggested fix: when a turn end is held for missing output files after an operator-started turn, send the agent a reminder or hand the pane back to the operator with a visible notice instead of waiting out the whole timeout.
Classify a timeout that follows an operator interrupt as resumable, not as `producer-hard-failure`.

Provenance: task `transient-tripout-overpull`, from `loom-halt.md`.




## From issue #255

Imported from [issue #255](https://github.com/Knatte18/loomyard/issues/255).

The Describe step wrote a pull request body that names lym, LoomYard and another task, and the run's docs carried the same kind of internal names.
Code-Review approved the diff, and PR-Review raised the leak afterwards as a send-back with three findings: one BLOCKING (the pull request body names lym internals), one MEDIUM (two document sections describe lym internals), one NIT.
The same class of leak had already been fixed by hand-written commits earlier in the run, such as a rewrite of a document to describe test tiers by what they run instead of through the gate.

This is abnormal because the output goes into a repository that does not know lym: a pull request body and repository documents should not name lym, LoomYard or other tasks.
Neither the Describe nor the Code-Review instructions caught it, so the leak cost a PR-Review send-back and a second Code-Bouncer, Code-Burler-Review and Code-Burler-Fix cycle.

What to do:
- Tell the Describe step, and any step that writes repository documents, not to name lym, LoomYard, gates, or other tasks in a pull request body or a repository file.
- Have Code-Review check documents and the pull request body for such names, so the finding comes before approval and not from PR-Review.

Provenance: task `dead-code-and-timing-docs`; notes `pr-review-send-back-1.md`, `loom-halt.md` (history only).
