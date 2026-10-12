# Bench the chair-and-advisors Discussion against the solo writer before switching it on

`discussion-multi-llm-bench` · Note · enhancement, loom, stencil, config

## Seen

`discussion-multi-llm` adds a second Discussion producer: a chair that writes, with advisors that study the code and answer its design questions live.
The default keeps the existing solo Discussion-Write, so landing it changes no run.

## Idea

Before any hub selects the chair-and-advisors producer, bench it against the solo writer on the same tasks: Discussion-Review rounds and findings, parent-review outcome, token cost per seat, and wall time.
Then the operator watches it in action on at least one live run before it is switched on.

## Operator decisions (2026-10-10)

- Models for the trial: the chair on `opus[medium]`, one advisor on `sonnet[high]`.
- Not in production until the bench and a watched run have shown it holds.

The plan to retire the single producer moved to `seats-replaces-singlellm` (2026-10-12).

## Bench design (operator, 2026-10-11)

Runs after `seats-chair-first` has landed and is deployed, so the chair starts first and asks in rounds.
A fresh agent run from the prime drives it; neither loom nor darn fits, since the output is a report, not a change.

- Reference: `webster-stability`, whose Discussion solo `fable[high]` wrote on base 800a0e892.
  Its first draft (records repo, `archive/webster-stability/*`, commit 82e8b0f9), its final discussion after three Discussion-Review rounds (e77a1372), and each round's findings are the answer key; `loop-small-bugs` (base 292bec755) is the fallback.
- Configurations, at most one advisor:
  - seats (chair, advisor): (`opus[high]`, `opus[medium]`), (`opus[high]`, `sonnet[high]`), (`opus[high]`, `sonnet[medium]`), (`opus[medium]`, `sonnet[medium]`);
  - seats with N=0: `fable[medium]`, `opus[high]`, `opus[medium]`;
  - control: the single producer on `opus[medium]`, against seats N=0 on `opus[medium]`.
- Method: one bench pair per configuration, reset to the base, models in the pair's own `loom.yaml` (`discussion_producer`, `discussion`, `discussion_advisors`), `lyx loom step` through Discussion-Write only, then the decision record and support log copied out.
- Order: one DualDW run first, (`opus[high]`, `sonnet[high]`), which the operator watches; the rest run in parallel once that one holds.
- Grading: each reference finding the Bouncer kept, avoided or repeated; decision overlap with the final discussion, new and missing decisions; writer tokens and wall time (`tools/tokencount`); how many batches the chair sent and whether it used the answers.
- Report: `docs/research/discussion-bench.md`.
- Second round (operator, 2026-10-11): a separate bench round with two advisors, such as chair `opus[high]` with advisors `opus[medium]` and `sonnet[high]`, testing whether a broader first draft beats a later Burler review-and-fix round in strength and speed.
  It should also measure how many Burler rounds each draft needs to converge and the total time and tokens to the final discussion, not only the draft against the answer key.
  A co-chair variant (see `discussion-advisor-mode`) is a candidate for the same comparison, including its blind compared-lists form (operator, 2026-10-12): the co-chair explores without the chair's questions and the two decision lists are compared.
- The operator confirmed (2026-10-11) that the bench agent may create and tear down the bench pairs, add a temporary board note per bench slug holding a copy of the reference task body (the writer reads it with `lyx board get`) and remove them after, and approve each bench pair's Discussion parent-review gate itself.

## Seen in the first run (2026-10-11)

The operator, watching `dbench-ohsh`, saw that the chair-advisor back and forth takes time and remarked that each batch should carry many questions.
The first round runs on the current chair stencil, unchanged, to learn whether the advisor setup is worth having; if it is, a candidate change is for the chair to send every question it has in batch 1 and only follow-ups after.

## Was `chair-blind-questions`

## Seen

`contracts/stencils/loom/loom-template-discussion-chair.md` tells the chair to send each question with its recommended answer and its alternatives to every advisor.
The advisor then reads the chair's pick before it studies the question, so its answer is anchored on the chair's options and lean, and it rarely finds an option the chair did not list.

## Idea (operator, 2026-10-11)

- The chair still forms each question with its options and its own recommended answer, but keeps them to itself.
- It sends the advisors the bare question only, so each advisor explores and answers on its own.
- The chair then weighs the advisors' answers against its own pick; an answer outside its options counts as a new option.
- Questions must therefore be open, never leading: a question must not name or hint at the chair's preferred answer.

## Open

- Whether the ledger records the chair's private pick before the advisors answer, so a review can see where an advisor changed the outcome.
- Whether the advisor stencil (`loom-template-discussion-advisor.md`) needs a line saying it gets no options and should name the alternatives it sees.

## Was `discussion-advisor-mode`

Its variants are candidate arms of the bench's second round (triage, 2026-10-12).

## Seen

The chair-and-advisors Discussion (`discussion-multi-llm`) sends every question batch to every advisor, by the operator's PR-Gate ruling on PR #504 (2026-10-10).
The first draft instead gave each advisor its own area of the design, which narrows each advisor's view but spreads the study.

## Idea

Support both, selected per hub in `loom.yaml` (for example `discussion_advisor_mode: all | areas`, default `all`), with the value passed to the chair stencil as a marker.
Decide after `discussion-multi-llm-bench`, which can compare the two modes on the same tasks.

## Operator-named areas

The operator also wants to choose which area each advisor studies (2026-10-10), rather than leaving the split to the chair.
One shape: each `discussion_advisors` entry carries a model-spec plus an optional area, for example `{model: sonnet[high], area: error-handling}`, and the area names a lens text the advisor stencil includes, the way Burler's review fans name lenses (`correctness`, `error-handling`, `test-gaps`).
An entry without an area answers every question; an entry with one still sees every batch but answers from its lens.
This could replace the `all | areas` switch: the per-entry area is the selection.

## Co-chair (operator, 2026-10-11)

An alternative to an advisor: one seat is a co-chair that does its own exploration of the code and the task rather than only answering the chair's batches, so the draft gets a second independent study.
How the co-chair's findings reach the chair, and who writes the decision record, is open.

## Blind exploration and compared lists (operator, 2026-10-12)

From an outside review the operator brought in of why the chair-and-advisors Discussion did poorly: whoever asks the questions owns the solution space, so an advisor is only as broad as the questions it gets; dropping the recommendation helped, but the questions and options still anchor it.
Independent thinking comes from different information and a different job, not from a different model.
Three steps beyond `chair-blind-questions`, `discussion-premise-cites` and the areas above, all building on the co-chair:

- Blind parallel exploration: the co-chair gets only the task and the code, never the chair's questions, and writes its own list of design decisions with answers; the two lists are then compared, and the value lies where they disagree or where one has a decision the other missed.
  The review named this the one step to take if only one is taken, since it attacks both the anchoring and the breadth.
- Question critic: before answering, the second agent critiques the questions: which decisions are missing, which questions are false choices between two options, and what they assume that is not settled.
  It gets bare questions only, never options (`chair-blind-questions`), so it has nothing to choose among.
- Disagreement escalates: a decision where the agents disagree, or where uncertainty is high, is recorded as an explicit assumption or sent to the parent, so the decisions that matter are the ones surfaced.

The blind compared-lists variant is a candidate arm in `discussion-multi-llm-bench`'s second round.

## Lens advisors as forks of the chair (2026-10-11)

The operator compares lens advisors with Burler's review fan, whose lens forks each review from one focus.
Burler's lenses are forks of the reviewer: they inherit its context and cached prefix and run on its model.
Seats advisors are fresh sessions on their own models, and each re-reads the code; in the first bench run (`dbench-ohsh`) the advisor's cache reads exceeded the chair's.
A variant: fork the lens advisors from the chair once it has read the task and code, so each lens starts with the chair's study at little cost, at the price of running on the chair's model.
Fresh-session advisors give breadth through different models, chair forks through different focus; a later bench can compare the two.
Lenses need a producer change either way: today the engine and chair stencil send every batch to every advisor with no focus.

## Board advisor in place of parent review (2026-10-11)

The operator asks whether an advisor with a board lens could take over Discussion's parent review, whose job is mostly checking the draft against everything else planned on the board.
Parent review also checks the draft against operator decisions made in the orch chat but not yet on the board, and against runs in flight; the one parent-review reject on 2026-10-11 (`per-role-models` round 1, seed model) was of that kind, which no advisor could have caught.
Proposal: put a board-lens advisor in the second bench round and compare what it catches with what parent review catches before making parent review optional.

## Advisor answers (operator, 2026-10-11)

Advisors write too much.
When an advisor agrees with the chair's pick, it says so in one line with no reasoning, since "I agree because ..." is clutter for the chair.
Every extension, correction or departure is explained.

## When (operator, 2026-10-11)

Advisors with different roles is a later enhancement.
The first DualDW bench runs one advisor at most, answering every batch as a plain assistant.
