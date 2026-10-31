# A status-bar button opens a short-lived board agent, so board talks need no orch

`board-agent-button` · Note · feature, reed, board, shuttle, orch

## Seen (operator, 2026-10-31)

Adding, changing or removing board entries is done today by talking to the orch, a long-lived session that carries every run's notices, escalations and rules, which is heavy for a talk about one or a few notes.

## Idea

A reed status-bar button of its own, beside the buttons of note `reed-status-buttons`, opens a short-lived agent for board work.
- The button opens an ordinary agent, as `lym shuttle open` does, with a skill loaded that makes it a board agent; no new role, so the same way can make other kinds of agent later (operator, 2026-10-31).
- The agent reads the board and the code it needs, talks one or a few cases through with the operator, writes, merges or removes notes, promotes notes to tasks and releases tasks on the operator's named go-ahead (operator, 2026-10-31), and exits.
  The board holds its state, so it needs no handoff and no context cycle.
- `ly:board` holds the mechanics of the board only; the policy, today the orch role's "The board" section (one finding per note, discuss before writing, a body holds the current design only, propose bundles and wait for approval, release only on a named go-ahead), moves into a skill of its own that the button loads and the orch role points to, so the policy lives in one place.
  It does not go into `ly:board`, which run agents load to file notes.
