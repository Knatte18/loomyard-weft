# Redesign adopt on one FSM of run state, pushed to GitHub at all times

`adopt-redesign` · Task · feature, bug, batten, tender, dobby, orch, fabric, loom, cleanup, workspaces, shuttle · running · Worktree-Create

## Goal

Redesign how a hub and its runs move between machines, starting with `lym adopt`, which almost never works today despite its rework of 2026-11-11 (commits `0deeb9b9` to `90d11895`).
The operator's requirement (2026-11-15):

- The state of lym and of every run in a lym hub is a finite state machine, and its whole state is stored on GitHub at all times.
- The operator can end every run in a lym hub at any moment, shut everything down, and go on later,
  on this machine or on another one after pulling the git state.
- Today this fails: lym always drifts and complains about all sorts of things, so some state clearly lives outside git.

The Motif starts from the state map `docs/research/loom/lym-state-map-2026-11-15.md` (commit `20bde208`):
every piece of state lym keeps, where it lives and whether git holds it.

## Operator decisions (2026-11-15)

- Adopt runs in the foreground, in the operator's terminal, never as a detached background job.
- Nothing else happens in the hub until adopt is done: no run starts, no dobby pass, no orch action,
  until adopt has finished and every failing test it found is fixed.
- A machine cannot know what another machine holds uncommitted or unpushed; only what that machine has pushed is visible.
  Adopt works from pushed state alone and never asks about the other machine's local state.
- lym detects that it has not run for a while, such as after the machine slept or hibernated or was switched off.
  When it notices, it first checks whether ownership or any other pushed state changed meanwhile, before it acts on anything.

## Direction

- Define each run's states and transitions as one FSM whose current state is a committed and pushed record.
- Each transition writes and pushes its record before it acts, so a stop at any point leaves a state another machine can read and go on from.
- State that cannot live in git, such as a live tmux pane or a session transcript, is a cache that lym rebuilds from the pushed state,
  never a source a resume or an adopt depends on.
- With that in place, adopt becomes: pull, read the FSM state, and go on, with no reconciling step.

## Bugs this design must settle

- **Adopt fails (operator, 2026-11-12 and 2026-11-15):** much housekeeping around every use.
  The state map's section 5 lists the failures on record (A1–A14) with their error text;
  the 2026-11-15 adopt of `root` itself ended `done` with a green check.
- **The orch comes back as a new session (A6, A7):** on both recorded adopts the orch's resume was refused because a live process still held the previous session
  (`session e7be957b-… is held by live process …`), so lym launched fresh with the typed instructions carried over (`internal/shuttleengine/carryover.go`).
  The operator wants the orch to come back as a continuation of its previous session, never as a new one.
- **The owner machine has two spellings (A8):** the Neve orch watcher exited with `owned by machine osl46`, where earlier lines say `osl-1046`.
  Owner identity is the machine name, so a second spelling reads as another machine.
- **Left-over pairs stand forever:** the bench pairs `pwc-opus` and `pwc-fable` still stood three days after their bench landed (report `384d7cb7`).
  The root `loomyard.code-workspace` still lists them, so they show in the operator's VS Code.
  The kept pair of the cancelled raddle run `claude-account-weights` stands the same way,
  and the `rerun` pair from the advisor trial fails to respawn on every `lym tender resume`.
  Nothing tears down a pair that a bench or a cancelled run left behind, and nothing tells the operator it stands.

## After this

The re-parent verb (report `docs/research/loom/reparent-running-task-2026-11-12.md`) waits until adopt is settled.
