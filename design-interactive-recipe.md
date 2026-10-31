# An interactive recipe: the operator iterates with an agent in the run's own worktree until done

`interactive-recipe` · Note · feature, shed, batten, reed, shuttle

## Seen (operator, 2026-10-31)

Some work needs many rounds between the operator and an agent on the result itself, and a one-shot writer makes it badly: a README written as a teaching text (task `readme-rewrite`), the board's README (note `board-viewer`), and trying out the reed status bar's buttons (note `reed-status-buttons`).
Darn's writer is non-interactive, and loom's `discussion_interactive` only interviews the operator before anything is written.

## Idea

A new Shed recipe beside loom and darn: an interactive agent the operator iterates with in the run's own worktree.
Batten stays the same for it as for every recipe.

- Batten makes the worktree, opens the window and starts an interactive agent with the task's body as its brief; the agent waits for the operator in its pane.
- The operator and the agent go as many rounds as the operator wants; the agent commits as it goes.
- The operator says the work is done, and the run goes on as usual: an optional review perch, off by default, then Describe, Finalize and Land.
- The result is one of:
  - a change to the repository, such as the README or `docs/`, landed as an ordinary commit;
  - an entry on the board, written by the agent through `lym board`, in which case the run has no commits and lands empty, which lym already supports;
  - a trial, as `reed-status-buttons` needs, in which case the run can use a tmux server of its own, so that what it tries does not disturb the operator's strands.

## Open

- How the operator says done: a `lym` verb, a reed status-bar button, or both.
- The recipe's name.
