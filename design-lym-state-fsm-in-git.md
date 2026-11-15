# The state of every run is one FSM, pushed to GitHub at all times, so any stop can go on here or on another machine

`lym-state-fsm-in-git` · Note · feature, batten, tender, dobby, orch, fabric, loom

## The operator's requirement (2026-11-15)

- The state of lym and of every run in a lym hub is a finite state machine, and its whole state is stored on GitHub at all times.
- The operator can end every run in a lym hub at any moment, shut everything down, and go on later,
  on this machine or on another one after pulling the git state.
- Today this fails: lym always drifts and complains about all sorts of things, so some state clearly lives outside git.
  `lym adopt` is the clearest case: it almost never works (see the note `adopt-does-not-work`).

## Seen

- Machine-local state lives under `.lym/` in the prime, Bolt and each pair: the orch's `state.json`, dobby and tender records, the adopt record, coverage maps, launchers.
- Claude session transcripts live in `~/.claude` on one machine only, so a run moved to another machine cannot resume its session.
- A machine cannot know what another machine holds uncommitted or unpushed;
  only what that machine has pushed is visible.

## Idea

Map every piece of state lym keeps: where it lives, whether git holds it, and which reader breaks when it is missing.
Define each run's states and transitions as one FSM whose current state is a committed and pushed record.
Each transition writes and pushes its record before it acts, so a stop at any point leaves a state another machine can read and go on from.
State that cannot live in git, such as a live tmux pane or a session transcript, is a cache that lym rebuilds from the pushed state,
never a source a resume or an adopt depends on.
With that in place, adopt becomes: pull, read the FSM state, and go on, with no reconciling step.
This is tapestry-sized: it runs as a Motif first.
