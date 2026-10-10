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

The seats producer with an empty `discussion_advisors` list runs as the single writer does (same prompt in effect, outputs, gates and timeouts).
Once that is tested, the single producer is parked, since seats with N=0 covers it; the bench should include an N=0 run against solo to show they match.
