# [WS 005] One dobby daemon per beam owns and drives its runs

`umbrella-daemon` · Note · enhancement, batten, orch, shed, workspaces, loom, board

## Seen (2026-10-18)

- Every batten run is its own Go process in a tmux window, and its Run-Shed row waits on the child with no time limit (`internal/battenshed/doc.go`): it sends halt notices, retries the first verify failure, revives a dead driver's strands, and lands and tears down once the child is done.
- Nothing restarts those processes after a machine restart; `lyx batten run <slug>` resumes from disk, and each row is idempotent, but someone must run it.
- The orch already has a Go daemon of its own, the watcher (`lyx orch watch`).

## Operator view, not a decision (2026-10-18)

An orchestrator gets not only a long-lived thread but a daemon that looks after every batten run it owns (those whose parent is its worktree) and sets their next steps going; see `orch-split`.

## Agreed in discussion (2026-10-18)

- One daemon per umbrella (see `orch-any-worktree`), the orch watcher, replaces one waiting batten process per run.
- The daemon follows its runs' status files (fsnotify, with a periodic re-read as a fallback where file watching is unreliable, such as WSL and junctions); the child only writes its status, as today, and knows nothing of its owner.
- Creel only where an agent must be told: the daemon posts to its thread when something needs judgement.
- The same pass finds dead drivers, so after a restart only the daemons need starting, one per umbrella.

## Orch recommendation, not a decision

1. Step 1: the Run-Shed row checks the child once and exits while it still runs; the daemon re-runs `lyx batten run <slug>` for each run it owns, so notices, the verify retry, revival and landing stay batten's own code, run in short passes.
2. Step 2, optional and later: move logic from batten into the daemon.

Open: who owns a run whose parent has no running daemon, the same problem as a thread that is not running.

## Operator decisions (2026-10-18, later)

- Everything `orch-split` proposed at the hub level belongs here: a Go program in each beam drives the runs of that beam, so the thread the operator talks to gets only what needs judgement; the split into two LLM agents is settled by the daemon, with no second LLM.
  Its later steps: start ready tasks with their run settings, run the landing routine, settle mechanical stops by rule with a cold helper where no rule fits, and the landing check for per-worktree config below.
- A worktree must be settable as a beam, the worktree with a thread and a daemon (see `orch-any-worktree` for the create command and the convert verb); the word replaces "umbrella" for that worktree, while the group of tasks is the project.
- The daemon never starts with the machine: `lyx orch start`, with or without `--continue`, starts the daemons of every beam in the hub, which after a restart replaces running `lyx batten run <slug>` for each run in flight.
- The daemon outgrows the name `ORCHWATCH` (its name and creel sender are now set below).

## Operator decisions (2026-10-18, beam and orch apart)

These replace the earlier wording above where they differ: the daemon is not the orch watcher, and `lyx orch start` does not own the daemons.

- A beam is a worktree plus its Go daemon, and is its own module, `lyx beam`.
  The daemon starts ready tasks, follows their batten runs, lands them and cleans up; it holds no LLM.
- An orch is only an LLM with a role: interactive and long-lived, hence its own watcher for context cycling.
  An orch can be attached to a beam, but a beam runs its tasks with no orch at all.
- What a beam's daemon cannot settle escalates upward: through the arbiter first, then to the beam's orch, or, when the beam has none, to the parent beam's orch, up to the prime.
- `lyx beam start` starts the daemons of every beam in the hub, and `lyx orch start` calls it first, so one command still starts everything; never at machine start.
- The orch watcher and the beam daemon are two different programs.
  The daemon is named `dobby`, after the loom mechanism that lifts the shafts by pattern, since "beam" names the worktree.
  As a creel sender it is the program `DOBBY`: `ly:DOBBY` for the root beam and `ly:<slug>:DOBBY` for another beam.
- Beam and dobby are one module: `lyx beam` holds the commands (`create`, `promote`, `start`), and dobby is that module's daemon, as the watcher is orch's.

## Who does what (operator decisions, 2026-10-18)

- The orch is the LLM the operator talks to: it discusses, writes notes and tasks on the board, and takes the escalations that need judgement (parent review, PR-Gate when the alignment agent escalates, design questions).
  It never starts, stops or cleans up a batten run.
- Dobby sets everything going and cleans up: it starts tasks, follows their batten runs, revives dead drivers, settles mechanical stops by rule, lands and cleans up, and hands on what it cannot settle, first to the arbiter and then to the nearest orch.
  It writes the board only for statuses and the prune, and makes no design call.
- Batten drives one task's lifecycle as today and knows nothing of who owns its run.

## A start flag on tasks (operator decision, 2026-10-18)

The operator often plans ahead by putting tasks on the board with their `depends_on` set, so a task is no longer by definition ready to be spawned.
A task carries a boolean flag of its own that says it may start.
Dobby starts a task only when the flag is set and its `depends_on` are done.
The operator can flip the flag themselves, and dobby then starts the task with no orch involved.

Orch recommendation, not a decision: the flag is named `released`, defaults to false, and is flipped with a board verb such as `lyx board release <slug>`; the README shows it.

## Folded in from `orch-split` (2026-10-11 to 10-13)

### Seen (operator, 2026-10-11)

The operator follows the orch from the phone through Remote Control, but so much comes in to the orch (batten notices, parent-review requests, escalations, PR-Gate reviews, landing routines) that the conversation is hard to follow.

### Idea (operator, 2026-10-11)

Split the hub orchestrator into two agents:

1. The run driver, the repo's upper driver: it reads the board, starts batten runs, and takes the escalated questions (parent reviews, review escalations, PR-Gate, landings).
   The operator can talk to it, but should not need to.
2. The conversational orch: the agent the operator talks to.
   It discusses problems with the operator and updates the board.

### Open

- How a question the run driver cannot settle reaches the operator: through the conversational orch, or as a notice of its own.
  The operator can keep both agents on Remote Control and gets a notification from the run driver when it needs input (operator, 2026-10-11).
- Who writes what on the board: the run driver only claims, sets statuses and prunes, while scope edits and notes stay with the conversational orch, or both write freely.
- How an operator decision reaches the run driver (start this bundle, change a hub config), and how the run driver learns the operator's settled decisions it needs at parent review.
- How much of the run driver is an LLM at all: most of the run loop is mechanical, as loom's driver now is; see `orch-owns-the-loop`.
  Starting a batten run on each ready board task (kind task, `depends_on` cleared, not claimed) needs no LLM (operator, 2026-10-11); nor does the landing routine.
  What the orch adds by hand today would need a home: per-task loom settings such as `review_max_bounces: 2` for a small task (a board field), and a cap on concurrent runs.
- `runs-overview` and `orch-token-spend` touch the same split.

### Idea: the run driver as a Go program (operator, 2026-10-12)

Perhaps the part split out of the orch is a Go-driven program: Go handles most of it on its own (starting ready tasks with their run settings, landings) and brings in the orch only when something goes wrong.
The same shape as loom today: the Go shed engine steps the run on its own (`lyx shed step --until-stop`), and the LLM driver above it wakes only when shed stops on something it cannot pass.
Per-task run settings (`config-layers-run-settings`) carry a task's review budgets and step models, so a start needs no LLM to set them.

### Two tiers, as in `go-driver-assistant` (2026-10-12)

`go-driver-assistant` (running) builds this shape for one loom run: a Go driver that settles stops by rule, a cold cheap assistant (the tackler) spawned only at a stop no rule settles, and the arbiter behind it.
The same two tiers fit the repo level:

1. A Go run driver: starts ready tasks with their `run_settings`, runs the landing routine (Land row, prod, intake), and posts what it cannot settle.
2. A spawnable assistant, cold per stop, never kept warm: it reads the stop and answers with a verdict or hands the question to the conversational orch.
   Scope and design choices always go to the orch.

Evidence from the loom level, a driver count over 98 runs (2026-10-05..12, 392 stops):

- plain relaunch 12 %; mechanical fix a Go rule could do 52 %; repair that needed an LLM 0 %;
- arbiter called 5 % (11 real attempts, all `resume`); relayed to the orch 22 %; other 10 %;
- drivers lived 370 h, 93 % of it idle.

The agent that ran the count estimated a cold helper would be needed at about 1 stop in 12 runs once Go takes the mechanical stops.
Nobody has counted the hub level yet: the orch's parent reviews, escalations, PR-Gates and landings by hand would need the same split before the Go share is known.

### A landing check for the Go run driver (operator, 2026-10-13)

At landing, loom warns that a task branch changed per-worktree config files (`_lyx/config/*.yaml`), which landing does not carry to the parent, and asks the parent to re-apply each change meant for it.
`logging-cleanup` raised it for a new, empty `_lyx/config/loom.yaml`: nothing to re-apply, and the orch found that only by diffing the file by hand.
The operator sees that check as a job for the Go run driver:

- parse the file at the fork base, the task tip and the parent tip, and compare keys;
- stay silent when the tip changes no key (an empty file, comments only) or the parent already holds the tip's value;
- otherwise name each changed key with its base, tip and parent values, so the parent knows exactly what to re-apply.

