# Beams: one lyx beam start for dobby and orch, the beam window, and machine ownership by project

`beam-start-and-window` · Task · enhancement, orch, workspaces, batten, reed, board

Depends on: [`umbrella-anchor`](design-umbrella-anchor.md)

## Seen (operator, 2026-10-25)

`lyx orch start` does four jobs: it starts every dobby this machine owns, reconciles the hub's branches against origin, ensures the overview window, and only then starts the orch session and its watcher.
Dobby's own verbs sit under `lyx beam` (`lyx beam start`, `lyx beam status`), and `lyx prime` names the top beam although the verb runs in any beam.
`--handoff` and `--adopt` are never used; `--continue` is used often and is long to type.
The command surface carries long-form shortcuts beside the short ones and a top-level `lyx start` for `lyx loom start`, which nobody runs since batten starts every loom run.
The overview window, the `runs` strand running `lyx batten overview`, shows the orch's context line, the runs in flight, and since `dobby-overview-window` the worktree's dobby with its runs, so the runs show in two places; its name says neither that it belongs to a beam nor what it covers.
Ownership is recorded per task and for the prime (`lyx adopt <slug>`, `lyx adopt --prime`), although the prime is a worktree, local to one machine like every beam.

## Design (operator, 2026-10-25)

A beam, the prime included, is one unit of two parts that always run together: its daemon, dobby, which starts, drives and lands the beam's runs, and its LLM, the orch, which handles stops and questions and is always there to talk to.
Dobby already sends its notices to the orch, and an idle orch spends nothing until a message wakes it.
Both run on the machine that owns the beam.

### Starting a beam

- `lyx beam start [slug]` brings a beam up, the current worktree's beam with no slug: it starts the dobby, reconciles the branches, ensures the beam window, and starts the orch with its watcher.
  The orch resumes its recorded session when one exists, else starts fresh from the last handoff note, else from the start stencil.
  A recorded session that cannot be resumed falls back to a fresh start from the note, never to an error.
- It starts only that beam; `lyx beam start --all` starts every beam this machine owns, such as after a restart, so the prime behaves like any other beam.
- It refuses in a folder that is no beam, a task worktree naming the beam it belongs to and its path; Ownership below says when it refuses on another machine's project.
- It is idempotent: a live dobby or orch is left alone, and only a missing part is started, a new orch resuming its recorded session.
- `lyx s` attaches at once, so the operator never sees its output on success: what runs shows in the beam window, and a refusal attaches nothing and leaves its error on the terminal.
- `lyx beam stop [slug]` stops both, and `lyx beam status` shows both.
- A fresh orch comes from `lyx orch refresh`.
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw` and `rules`.
- Removed: `lyx orch start` with `--continue`, `--handoff` and `--adopt`; `lyx orch cleanup`, since `lyx cleanup` already removes old handoffs; `lyx prime`; `lyx start`.

### Shortcuts

Shortcuts are short forms only:

| Shortcut | Runs |
|---|---|
| `lyx s` | `lyx beam start`, then `lyx reed attach` |
| `lyx a` | `lyx reed attach` |
| `lyx p` | `lyx tender pause` |
| `lyx r` | `lyx tender resume` |

The long forms `lyx attach`, `lyx pause` and `lyx resume` are removed.

### The beam window

The overview window becomes the beam's own window, named `beam: <slug>`, `beam: prime` in the prime, with one section per part of the beam:

- Dobby: alive, `max_runs`, each run it drives with its state and stage, and each released task it has not started with what it waits on.
- Orch: its context reading against the soft threshold and the hard cap, and its cycle state, as the context line shows today.

Each run shows once, in the dobby section.
A beam's window shows its own orch and the orchs of every beam below it.

### Ownership

A worktree, and so a beam, belongs to one machine; the branches are shared, and the board's entries are common to every machine.
The board itself has no owner: ownership sits on projects.

- The top level of the board is a project of its own, the root project, driven by the owner machine's prime.
- Each project is owned by one machine and driven by its beam there; two machines can work at once on projects they each own, each with only its own project checked out.
- A task has no owner of its own: it follows its project, whose owner's dobby starts and drives it.
- `lyx adopt <project>` moves a project to this machine with its tasks and runs in flight, and `lyx adopt root` replaces `lyx adopt --prime`; only projects are adopted, never a single task.
- There is no adopt of a beam.
- `lyx beam start` refuses on a beam whose project another machine owns, naming `lyx adopt`.
- `lyx adopt` of a project starts its beam, dobby and orch, on the adopting machine.
- An adopt moves that project alone: a beam project nested under it keeps its own owner, so adopting root moves root's own tasks and never the beam projects under it.
- Landing a beam into its parent is the parent beam's job: the parent's dobby runs the landing and cleans up after it, so a machine lands a project only while it owns the parent as well.
  Example: with G and H beam projects under root and everything on Neve, OSL-1046 adopts G and finishes it, then adopts root to land G; H stays on Neve throughout.

### Landing a beam

- `lyx beam land <slug>` stops the beam's orch and its watcher itself instead of refusing while they live; its teardown already ends the beam's dobby.
- When a beam's last member lands, its dobby sends its orch a notice that the beam can land, naming `lyx beam land <slug>` in the parent worktree; when another machine owns the parent project, the notice says so and names `lyx adopt` of the parent.
- `lyx beam land` refuses on a machine that does not own the parent project, with the same way forward, so no landing starts that no dobby drives.
