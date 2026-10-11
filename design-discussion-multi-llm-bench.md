# Bench the chair-and-advisors Discussion against the solo writer before switching it on

`discussion-multi-llm-bench` · Note · enhancement, loom, stencil

## Seen

`discussion-multi-llm` adds a second Discussion producer: a chair that writes, with advisors that study the code and answer its design questions live.
The default keeps the existing solo Discussion-Write, so landing it changes no run.

## Idea

Before any hub selects the chair-and-advisors producer, bench it against the solo writer on the same tasks: Discussion-Review rounds and findings, parent-review outcome, token cost per seat, and wall time.
Then the operator watches it in action on at least one live run before it is switched on.

## Operator decisions (2026-10-10)

- Models for the trial: the chair on `opus[medium]`, one advisor on `sonnet[high]`.
- Not in production until the bench and a watched run have shown it holds.

## Retiring the single producer (operator, 2026-10-10)

The seats producer with an empty `discussion_advisors` list covers the single writer's job: one chair prompt for every advisor count says "use your advisors if you have any; with none, decide each question yourself as you judge best", with no conditional rendering.
Once that is tested, the single producer is parked, since seats with N=0 covers it; the bench should include an N=0 run against solo to show they match.

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
  A co-chair variant (see `discussion-advisor-mode`) is a candidate for the same comparison.
- The operator confirmed (2026-10-11) that the bench agent may create and tear down the bench pairs, add a temporary board note per bench slug holding a copy of the reference task body (the writer reads it with `lyx board get`) and remove them after, and approve each bench pair's Discussion parent-review gate itself.
