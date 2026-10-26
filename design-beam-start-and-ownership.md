# Beams: one lyx beam start, machine ownership by project, and a beam's life as a pawl run

`beam-start-and-ownership` · Task · enhancement, orch, workspaces, batten, reed, board · running · Parent-Align

## Seen (operator, 2026-10-25)

`lyx orch start` does four jobs: it starts every dobby this machine owns, reconciles the hub's branches against origin, ensures the overview window, and only then starts the orch session and its watcher.
Dobby's own verbs sit under `lyx beam` (`lyx beam start`, `lyx beam status`), and `lyx prime` names the top beam although the verb runs in any beam.
`--handoff` and `--adopt` are never used; `--continue` is used often and is long to type.
Ownership is recorded per task and for the prime (`lyx adopt <slug>`, `lyx adopt --prime`), although the prime is a worktree, local to one machine like every beam.

## Design (operator, 2026-10-25)

A beam, the prime included, is one unit of two parts that always run together: its daemon, dobby, which starts, drives and lands the beam's runs, and its LLM, the orch, which handles stops and questions and is always there to talk to.
Dobby already sends its notices to the orch, and an idle orch spends nothing until a message wakes it.
Both run on the machine that owns the beam.

### Starting a beam

- `lyx beam start [slug]` brings a beam up, the current worktree's beam with no slug: it starts the dobby, reconciles the branches, ensures the beam's overview window, and starts the orch with its watcher.
  The orch resumes its recorded session when one exists, else starts fresh from the last handoff note, else from the start stencil.
  A recorded session that cannot be resumed falls back to a fresh start from the note, never to an error.
- It starts only that beam; `lyx beam start --all` starts every beam this machine owns, such as after a restart, so the prime behaves like any other beam.
- It refuses in a folder that is no beam, a task worktree naming the beam it belongs to and its path; Ownership below says when it refuses on another machine's project.
- It is idempotent: a live dobby or orch is left alone, and only a missing part is started, a new orch resuming its recorded session.
- A new shortcut `lyx s` runs `lyx beam start`, then `lyx reed attach`; it attaches at once, so the operator never sees its output on success: what runs shows in the overview window, and a refusal attaches nothing and leaves its error on the terminal.
- `lyx beam stop [slug]` stops both, and `lyx beam status` shows both.
- A fresh orch comes from `lyx orch refresh`.
- `lyx orch` keeps `stop`, `status`, `distill`, `refresh`, `withdraw` and `rules`.
- Removed: `lyx orch start` with `--continue`, `--handoff` and `--adopt`; `lyx orch cleanup`, since `lyx cleanup` already removes old handoffs; `lyx prime`; `lyx beam promote`, since a beam is made from its project, never from a running task.

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

### A beam's life is a batten run

A beam is made, landed and torn down by an ordinary batten run over its project entry, the same batten as for loom and darn: Worktree-Create, Run-Shed, Land, Worktree-Teardown.
Only the recipe Run-Shed runs differs: the `pawl` recipe, today's `beam` recipe renamed, since `beam` names the worktree; a beam project's entry carries `recipe: pawl`.
A pawl is the catch that holds a loom's warp beam until the weaver releases it, as Beam-Await holds the beam until `lyx beam land`.

| Row | Does |
|---|---|
| Beam-Up | marks the worktree as a beam, then starts its dobby and orch |
| Beam-Await | waits while the beam's own dobby runs its members, until `lyx beam land` ends it |
| Parent-Align | merges the parent into the beam, as loom's row does |
| Code-Review perch | Code-Bouncer, Code-Burler-Review and Code-Burler-Fix, loom's perch, reviewing the beam's diff after the merge |
| Beam-Describe | writes the landing description, as today |
| Finalize | merges the beam into its parent from inside, as for loom and darn; the conflict session resolves a conflict and an overlapping merge goes on to the landing gate, with no reroute |
| Beam-Down | stops the beam's orch, its watcher and its dobby |

- `lyx beam create <slug>`, run from the parent worktree, marks the project entry a beam with its owner and starts its batten run at once; it no longer makes the worktree itself.
- `lyx beam land <slug>` ends Beam-Await; it refuses while the beam has open members or member worktrees, and on a machine that does not own the parent project, naming `lyx adopt` of the parent.
- The separate landing run of today's `lyx beam land` goes.
- A beam's landing is the heaviest of the three recipes, since a project is large: where loom and darn only resolve a moved parent in Finalize, pawl aligns with the parent and reviews the result before it lands.
- Batten's start guard lets a beam project start, where it now refuses every project entry but a beam ready to land.
- When a beam's last member lands, its dobby sends its orch a notice that the beam can land, naming `lyx beam land <slug>`; when another machine owns the parent project, the notice says so and names `lyx adopt` of the parent.
