# Split the orch into a run driver and a conversational orch

`orch-split` · Note · feature, orch, batten

## Seen (operator, 2026-10-11)

The operator follows the orch from the phone through Remote Control, but so much comes in to the orch (batten notices, parent-review requests, escalations, PR-Gate reviews, landing routines) that the conversation is hard to follow.

## Idea (operator, 2026-10-11)

Split the hub orchestrator into two agents:

1. The run driver, the repo's upper driver: it reads the board, starts batten runs, and takes the escalated questions (parent reviews, review escalations, PR-Gate, landings).
   The operator can talk to it, but should not need to.
2. The conversational orch: the agent the operator talks to.
   It discusses problems with the operator and updates the board.

## Open

- How a question the run driver cannot settle reaches the operator: through the conversational orch, or as a notice of its own.
  The operator can keep both agents on Remote Control and gets a notification from the run driver when it needs input (operator, 2026-10-11).
- Who writes what on the board: the run driver only claims, sets statuses and prunes, while scope edits and notes stay with the conversational orch, or both write freely.
- How an operator decision reaches the run driver (start this bundle, change a hub config), and how the run driver learns the operator's settled decisions it needs at parent review.
- How much of the run driver is an LLM at all: most of the run loop is mechanical, as loom's driver now is; see `orch-owns-the-loop`.
  Starting a batten run on each ready board task (kind task, `depends_on` cleared, not claimed) needs no LLM (operator, 2026-10-11); nor does the landing routine.
  What the orch adds by hand today would need a home: per-task loom settings such as `review_max_bounces: 2` for a small task (a board field), and a cap on concurrent runs.
- `runs-overview` and `orch-token-spend` touch the same split.
