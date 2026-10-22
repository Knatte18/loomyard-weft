# Support log — run-process-ownership

## Interview

### Turn 1 — exploration (autonomous mode, no operator)

Read the board entry and the related entries `batten-cancel-verb`, `spawn-audit-log`, `umbrella-daemon`, `fabric-remote-async` and `verify-watchdog`.
Read the code that starts and ends run processes: `internal/proc` (`StartTied`, `KillTree`, named jobs, `Detach`), `internal/pairteardown` (quiet rule, loop end, local mode, origin seams), the detached push and board-sync spawns, the loop pid record, the Go driver section of `internal/loomcli`, the pause record and network gate, `internal/tenderengine`, hibernate confirmation and shuttle's background-work rules.
Read the three commits since the incident's binary that settle parts of note `orphan-run-processes`: `e610f9250` (origin reconcile, records replay, visible push failures), `0d953f140` (owner machines, `lyx adopt`) and `fac869bb3` (a pause holds prompts, verify commands and new batches).

Findings that shaped the design:

- Detached children (`proc.Detach`, a new session) are recorded nowhere except the step loop's pid file, so nothing but `ps` finds a driver, batten process or push.
- Under a pause, every lyx process waits at the network gate with no deadline, and the Go driver and batten wait in `WaitUncovered`; those are the parked processes of the incident.
- `fabric remove`'s quiet rule reads a held driver lock as busy whatever the pause, while the reconcile's `LivePair` seam already reads a paused slug as not live.
- A gated shuttle run never treats a waiting turn end as an arrival, so a reviewer's shells hold the row however long they run (issue #796).
- Hibernate confirmation reads any background work as busy.

- The 2026-10-20 incident's survivors after `reed down` and the pause were the Go driver, the step loop, batten-side processes and the detached pushes; D4 ends exactly these.
- Issue #796's review passed its gate in 7.5 minutes and then waited 21 minutes on two `find /` shells; D11 finishes it at that turn end.

### Turn 2 — design decisions, all auto-picks

Questions and picks are in the question ledger; the decision record holds the result, renumbered D1 to D11 after the lens review.

### Turn 3 — lens review

All seven forks returned (interactions, existing-paths, bound-breaker, walkthrough, goal-scope, board-fit, leanness).
Duplicates are merged; each line names the lenses that raised it.

- remove under a hub pause blocks at its own reconcile and pushes (interactions, walkthrough) — fixed: remove's own calls pass the gate under any pause (D5).
- hub-scope records always read as orphans, so reap would kill board syncs and prime pushes (interactions, bound-breaker) — fixed: the hub scope is never an orphan (D8).
- the paused-run skip kills a step mid-merge, "a paused driver does nothing" is false (bound-breaker) — fixed: the run-lock wait stays, and only a holder waiting at the gate is ended (D5, PATTERN text).
- Finalize Done reading kills batten mid-Land (interactions, walkthrough, bound-breaker) — fixed: counts only once batten no longer reads running (D5).
- `reed down` gains an unbounded kill of a live run (interactions, walkthrough, bound-breaker) — fixed: same readings as D5, refuses a running unpaused run (D4).
- the reconcile's `Teardown` seam returns busy before `End` runs; skip placement inconsistent with the PATTERN text (interactions) — fixed: the skip is a `pairteardown` rule the seam shares (D5 note, PATTERN text).
- owed pushes respawned only by `tender resume`, while other lifters call `Lift` (existing-paths, bound-breaker) — fixed: the spawn seam sits in the lift sequence (D6).
- the push child takes its pending flags before the network call, so "flag left in place" is false (existing-paths, walkthrough) — fixed: coverage is checked before the flag is marked, and a later covered call puts flags back (D6).
- a child past its first network call parks again (bound-breaker) — fixed: exits at any covered call (D6).
- hub-entry resume pushes for no pair, and a slug resume never respawns the prime push (existing-paths, interactions, walkthrough, bound-breaker) — fixed: every uncovered pair plus one prime push on every lift (D6).
- prime records pushes registered under the run's slug, killed by a pair's teardown (walkthrough) — fixed: registered under the hub scope (D2).
- board sync has no carrier for the no-wait mode (existing-paths, walkthrough) — fixed: a new hidden flag (D6).
- lift pushes for a run owned elsewhere (interactions) — fixed: excluded (D6).
- Windows: no start time and no ancestry, so End ends nothing or the wrong thing (bound-breaker) — fixed: End ends no recorded pid on Windows, owned as this task's platform bound (D3, Open risks); the record no longer hands it to `verify-watchdog`, whose scope does not cover it (board-fit).
- an unregistered older-binary process outruns the skip (bound-breaker) — fixed: driver-lock re-check after `End` (D4).
- the reconcile's `stopped` verdict unhandled; a reconcile that already removed the pair (bound-breaker, existing-paths) — fixed: `stopped` refuses, an already-removed pair reads done (D5).
- End races a spawn during the end (bound-breaker) — fixed: up to three passes (D3).
- the five-second grace per record could add up (bound-breaker) — fixed: one grace for all records (D3).
- descendants read after the first signal could reparent (walkthrough) — fixed: read before the first signal (D3).
- SIGTERM rationale does not hold for the step child (walkthrough) — fixed: rationale names pushes, syncs and commits, and the step child keeps its existing path (D3).
- D9 pane kill reaches hub-level processes from the prime's panes (bound-breaker) — fixed: pair sessions only, lyx and registered processes excluded (D9).
- D11 stall rule could cut the LLM driver's loop shell (existing-paths, bound-breaker, goal-scope) — fixed: limited to ungated runs; the LLM driver never enters `Wait` (`driverspec.go`), recorded with evidence; the remaining ungated risk is in Open risks.
- D11 "either kind" wording (walkthrough) — fixed: "an ungated run".
- the batten-record shield hides real orphans of removed pairs (existing-paths, walkthrough) — fixed: only a batten record reading `running` shields (D8).
- a starting batten reads as an orphan (interactions) — fixed: batten registers once its run record is seeded (D2).
- replaced-binary overlap in status (existing-paths) — fixed (D8).
- confirmation bound ambiguous when reasons interleave; orch rationale wrong (bound-breaker) — fixed (D10).
- `--remote` silently dropped in local mode (bound-breaker) — fixed: `remote_skipped_reason` (D5).
- parked driver chain under a pause only an assumption, a silent departure from the entry (goal-scope, leanness) — fixed: stated in D6 with the departure named.
- detached children of an interrupted call depart from the entry's Pdeathsig line silently (goal-scope) — fixed: departure named with its bound (D2).
- plain pause leaves shells running, unstated (goal-scope) — fixed: stated in D6.
- "one verb ends them all" split unstated (goal-scope) — fixed: auto-mode assumption names the split.
- Goal wording "keyed by the run's slug" (goal-scope) — fixed.
- `fabric-remote-async` contradiction by remove's reconcile (board-fit) — fixed: stated and resolved in D5 and Constraints.
- `spawn-audit-log` could build a second launch path (board-fit) — fixed: Scope names D2's sites as the ones it extends.
- `batten-cancel-verb` needs what an ended process does and what batten record is left (board-fit) — fixed: D4 states both, with the SIGTERM-handler fact.
- owned-elsewhere runs' processes on the machine left stay parked (board-fit) — fixed: new D7.
- `orch-strand-cycle` and dobby unnamed (board-fit) — fixed: plan-writer notes.
- a pending flag lost when `End` kills a holder on a live pair (interactions) — fixed: Open risks.
- the Webster fork context-bound idea left with no board home (board-fit) — rejected: the operator called it a separate idea, and filing it is not this task's call.
- leanness: duplicate assumption bullets, incident narrative in D4 and D10, D1's tracking line, generic test constraints, the module-doc list, the long Goal, the duplicate Windows risk — fixed: cut or moved to this Interview.

## Rejected alternatives

- Owner record as one JSON file per run, rewritten under a lock: concurrent spawners would contend on the lock, and a killed child cannot remove its own entry; one file per process pruned on read needs neither.
- Owner record inside the pair's `.lyx`: it goes with the pair's worktree, so an orphan of a removed pair would be invisible.
- Finding run processes only by scanning the process table (argv and working directory): argv shapes differ per verb and drift, and Windows cannot list processes; kept only as the fallback for unregistered orphans (D7).
- Linux cgroups or systemd scopes per run: a platform-specific mechanism with no Windows counterpart, and `systemd-run` is not guaranteed on every Linux host.
- A pause that ends the driver, step loop and batten, with resume restarting them: a step mid-wait on an interrupted agent would have to be reattached by every resume path; a parked driver is idle and cheap once it is owned and ended by every end verb.
- Killing every child with `Pdeathsig` when its spawning CLI call dies: detached pushes exist to outlive their spawner (`fabric-remote-async`), so the run, not the call, owns them.
- Signalling a self-registered process's group whatever it leads: a `lyx batten run` started from an agent's Bash tool shares the agent's group, and the group kill would end the agent.
- SIGKILL at once, as the loop end does today: a git push or commit killed mid-write leaves `index.lock`; SIGTERM lets git remove it.
- A pause that interrupts forks, or stops them from the agent picker: the operator rules out cutting a fork mid-batch as a default (2026-10-15).
- Hibernate confirmation treating forks as settled too: a fork still writes files and runs gates, and is never interrupted, so confirmation lets it finish.
- A config key for the hibernate fork bound: no caller asks to tune it; an engine constant matches the existing ten-minute bound.
- Shuttle reading a stopped background shell from the transcript to end the wait: the provider writes no reliable record for a task the operator stops by hand, and D10's arrival rule settles the review case without it.
- Reaping orphans in `lyx cleanup --apply` as well: one verb is enough, and status names it.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where does the owner record live, and in what shape? — auto-pick: one file per process under `<HubScratchDir>/runproc/<owner>/`, pruned on read (D1). Resolved.
- Q2 Who registers a process? — auto-pick: the spawner for a detached child, the role verb itself for a long-lived role; synchronous children stay tied (D2). Resolved.
- Q3 What is a board sync's owner? — auto-pick: the hub scope, since every board write spawns one and it serves no single run (D2). Resolved.
- Q4 How are processes ended, and which are spared? — auto-pick: SIGTERM, five seconds, SIGKILL; group only for a group leader; caller and ancestors spared (D3). Resolved.
- Q5 Which verbs end a pair's owned processes? — auto-pick: `pairteardown`'s session end, `lyx reed down` in a pair under D5's readings, the reconcile's owned-elsewhere pause, `lyx tender reap` for orphans, and the exported `runproc.End` for `batten-cancel-verb` (D4, D7, D8). Resolved.
- Q6 Does `tender pause` end the driver chain, or does `fabric remove` end it? — auto-pick: the driver chain stays parked and owned, and the end verbs end it; pushes and syncs stop parking (D4, D5, D6). Resolved.
- Q7 When does `fabric remove` skip its quiet wait's driver checks? — auto-pick: entry ended or owned elsewhere by the reconcile, slug covered by a live pause, or entry done or Finalize Done locally with batten no longer running; the run-lock wait stays (D5). Resolved after the lens review.
- Q8 How does a queued push under a pause become a record? — auto-pick: the marked child exits before marking its flag, or puts back its flags at a later covered call, and every lift spawns the owed pushes (D6). Resolved after the lens review.
- Q9 How does `tender status` find orphans? — auto-pick: registered processes of a slug with no pair and no running batten record, never the hub scope, plus unregistered lyx processes in a deleted hub directory (D8). Resolved after the lens review.
- Q10 Does reed's session end reach an agent's background shells? — auto-pick: yes, for pair sessions, non-lyx unregistered pane descendants read before the kill (D9). Resolved after the lens review.
- Q11 How does hibernate confirmation treat background work and the orch? — auto-pick: shells settled, forks waited on up to sixty minutes, the requesting orch left out (D10). Resolved.
- Q12 When does a reviewer's background shell stop holding its row? — auto-pick: a gated waiting turn end with only non-awaited shells is an arrival; an ungated shell-only wait past `BackgroundShellWaitMin` with outputs missing finalizes stalled (D11). Resolved after the lens review.
- Q14 Do `fabric remove`'s own network calls wait under a pause? — auto-pick: no, the remove process passes the gate for its own calls; its children do not (D5). Resolved, from the lens review.
- Q15 What does `End` do on Windows? — auto-pick: ends no recorded pid, since no start time identifies it; the step loop still dies through its named job; owned here as a platform bound (D3). Resolved, from the lens review.
- Q16 Does the machine a run left end that run's processes? — auto-pick: yes, at the reconcile's owned-elsewhere pause (D7). Resolved, from the lens review.
- Q13 Is the diverged Bolt and prime weft of the incident still in scope? — auto-pick: no, `e610f9250`'s reconcile and records replay settle it. Resolved.
