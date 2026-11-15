# Decision record — adopt-redesign

## Goal

Every run of a lym hub, and the hub itself, is one finite state machine whose state lives on GitHub at all times.
Each transition is committed and pushed before the run acts on it.
State that cannot live in git is a cache lym rebuilds from pushed state.
The operator can stop every run at any moment, and go on later on this machine or on another one after it pulls.
`lym adopt` becomes: hold the hub, take the owner, pull, read the state, check, and go on, with no reconciling of local state.

The design starts from the state map `docs/research/loom/lym-state-map-2026-11-15.md` and settles the failures its section 5 lists.

## Scope

In: the decisions below.

Out:

- The hub stop button (`hub-stop-button`) and the restart launchers (`machine-restart-shortcut`): they build on this design and stay notes.
- Claude account choice per worktree (`claude-account-pick-simple`); D10 restores into whatever config directory the claude engine resolves for the worktree.
- Carrying non-orch agent sessions between machines: a run's in-flight step starts again from its last pushed transition.
- A hub-wide operator pause shared between machines: `lym tender pause` with no slug and hibernate's pause stay machine-local, as today.
- The re-parent verb, which waits for this design to land.

## Decisions

### D1 — One run FSM, derived from tracked records by one package

A new leaf package `internal/runfsm` (new) declares a run's lifecycle states, their order, and the act rule.

A run's state is derived from tracked records alone: the board entry, the Bolt holds and owners (D5, D7), the batten run record, and the loom child run record.
A record is read from the local worktree, or from origin's branch when this machine has no pair for the run.
The states, highest precedence first, each taken when its condition holds and no earlier one does:

1. `landed`: the batten record `done`.
2. `cancelled`: a finished cancel (D3), the record saying whether the pair was kept.
3. `cancelling`: a cancel intent (D3) whose cancel has not finished.
4. `paused`: a run-scope pause intent (D3), or a shed state `paused`.
5. `halted`: the child run `blocked`, `failed` or `awaiting`.
6. `landing`: the child run at `Finalize` or a later row.
7. `running`: the batten record at `Run-Shed` or later.
8. `starting`: a batten record at a row before `Run-Shed`.
9. `queued`: a released task with an empty board status and no batten record.

A pair stands or not beside each state; a landed or cancelled run whose pair stands owes a teardown (D11).

The act rule answers, for one run on this machine, `act`, `wait` or `hand_over`.
Its inputs are the run's state, the owner read through `hubgeom.ParentOf` and the registry (D5), the Bolt holds (D7),
and two machine-local facts: a push owed with origin unreachable (D2), and this machine's pause record coverage.

- `hand_over` when another machine owns the run, unless the run is `landing` past its parent-side merge, which finishes here.
- `wait` when a hold covers it, a pause covers it, a push is owed and origin is unreachable, or the state is terminal.
- `act` otherwise.

Every verb or loop that acts on a run asks the act rule first, and every reader of a run's state takes it from `internal/runfsm` (new).
The board's `<state> · <row>` status is a display written from the FSM state; no decision reads it but `queued`'s empty status.

Rationale: the state map (section 6.4) finds six vocabularies that disagree on `paused`, `running`, `done` and the halted states.
One package over tracked records gives every machine the same state from the same pushed bytes.

### D2 — A transition pushes its records before the run acts

- A shed transition commits its records as today, the files D3 moves into tracked records included, then pushes them synchronously:
  both sides in a task pair, records only in the prime.
  The next producer starts only once the push has landed.
- The detached per-transition push spawn (`fabricengine.SpawnStatusPush` behind `internal/statuscommit/statuscommit.go`) retires,
  and so does fabric's pause deferral of a push, which has no detached push left to defer.
  `lym loom commit-records` keeps its synchronous push.
- A push that fails for a transient reason, such as an unreachable origin, leaves the run waiting at that transition with the push owed.
  It retries on a backoff under `PATTERN-daemon-wakeups` and starts no producer meanwhile.
  `lym beam status` shows the owed push and its last error, and the existing failed-push record and creel notice stay.
- A rejected push replays the records as today, then applies D4.
- A Bolt write that carries a decision pushes synchronously before its writer goes on:
  a run claim, the board `done` of a landing, an owner, a hold (D7) and the registry (D5).
  A board write that only displays a run status stays on the detached board sync.

Bound: no run acts while origin is unreachable.
Work an agent made inside a step and has not reached its transition yet is on this machine only;
a stop there loses nothing pushed, and the step runs again from its start.

Rationale: the operator requires the whole state on GitHub at all times, and today a push can lag or fail with nothing waiting for it (state map 1.3, 7).
A push costs two network round trips or more (`internal/fabricengine/pushclassified.go`), small beside an agent step.

### D3 — Run decisions move into tracked records; the rest is cache

These move from a pair's or the hub's `.lym` into tracked records under the run's `_lym/shed/<slug>/`:

- The cancel intent, in the batten record: when it started, the machine, and remove or keep.
  It replaces the cancel marker of `internal/battencli/cancelmarker.go`; the cancel's lock stays local.
  `lym batten cancel` commits and pushes it before it ends anything.
- A run-scope pause intent, in the batten record: who paused and when.
  `lym tender pause <slug>` writes it and `lym tender resume <slug>` clears it, each committing and pushing before it returns.
  Each also writes this machine's pause record entry as today, which the network gate and the turn interruption read.
- The pull-request record of `internal/pullrequest/record.go`, whole, in the pair's records; a write of it commits and pushes at once.
- The fixer attempt counts of `internal/landingshed/fixerattempts.go`, in the pair's records, committed with the transition.

A pause or cancel of a run another machine owns is refused, naming that machine, so only the owner writes a run's records.
A machine that becomes a run's owner writes its pause record entry from the run's pause intent at its catch-up (D6).

The origin-owed marker of `internal/battencli/resumereconcile.go` retires: under D2 and D6 no pass acts without origin.

Everything else run-related under `.lym` is a cache that a fresh machine starts without or rebuilds:
the merge record, the driver park marker and resume report, the batten per-row marks, the dobby's `beam/state.json`,
shuttle run directories, reed state, loom scratch, gate and verify records, the parent lock, the run process registry,
the creel journal, adopt check views, failed-push records, the hibernate request, and the pause record.
An absent cache reads as empty and refuses nothing.
A sent-notice memory lost this way can repeat a notice or a revive once on the new machine.

Tracked values that hold only on the machine that wrote them, the Claude session ids in webster's state and the pull-request record,
and absolute paths in step envelopes, are read as such:
a session id whose transcript is absent here starts that session fresh, and an absolute path is display only.

Rationale: the state map (section 7) lists these as the run-relevant state held only under `.lym`,
and `PATTERN-lym-files` already says a run depends on nothing that lives only there.

### D4 — A takeover entry in the run's own records hands the run over

- Adopt writes a takeover entry, naming the new owner machine, into each moved run's batten record and child record, each on its own branch, and pushes it (D8).
- When a run's records push is rejected, lym replays as today and reads the run's own record from the result.
  When that record holds a takeover entry naming another machine that this machine has not yet read, the run hands over:
  this machine pushes nothing more for it, ends its processes for the slug (D9),
  archives the local tips as the old owner's archive does (`fabricengine.ArchiveOwnerTips`), with uncommitted files first committed onto the archived tip,
  pushes the archive tags, and posts one notice naming the machine.
- Any other rejection, including commits of other runs on a shared records branch such as `main-weft`, replays and pushes as today.
- An adopter's own takeover push that is rejected replays over whatever origin holds and pushes again; it never hands over.

Bound: transitions the old owner pushed before the takeover landed stand, and the adopter goes on from them.
The old owner's step in flight runs to its next transition, which the takeover rejects; its result is archived under a tag, not lost and not on the branch.

Rationale: a takeover entry in the run's own record is a compare-and-swap per run that holds on records branches many runs share,
and needs no extra fetch per transition.

### D5 — A machine registry in Bolt gives each machine one identity

- `board.json` gains a top-level `machines` map: each registered name with the former names it replaced.
  Only `boardengine`'s owner setter writes it, pushing synchronously.
- A machine registers at its first owner write or `lym machine name`, copying the former names of its `machine.json`.
  A registration whose former name is another registered machine's current name is refused, naming that machine.
- `lym machine name` records the rename there, besides its sweep of owners (`internal/machinecli/set.go`).
- Every comparison of a recorded machine name with this machine goes through the registry:
  a recorded name that is a former name of a registered machine reads as that machine.
  `boardengine.OwnedElsewhere` takes the registry, and every owner write writes the current name.
- `internal/machinename` stays this machine's only name source.

Rationale: A8's `osl46` and `osl-1046` are one machine before and after `lym machine name`;
the former names live only in that machine's `machine.json`, so every other machine reads them as two.

### D6 — After an absence, lym catches up from origin before it acts

An absence is any of:

- a long-lived lym process, the dobby loop or the orch watcher, observing a sleep through `pauserecord.SleepWatch` over `pauserecord.SleepTolerance`;
- a worktree's dobby starting, as through `lym s`, `lym beam start` or a hibernate wake;
- a hand pass, which keeps the origin reconcile `PATTERN-owner-machine` gives it, now this catch-up over its run.

Before any act rule answer (D1) after an absence, the catch-up runs:

1. It fetches Bolt, replays local display commits as the board sync does, and reads the owners, the registry and the holds.
2. It fetches each local pair's branches and reads each run's record from the result.
3. For a run this machine owns, local commits on top of origin's tip are this machine's owed pushes, and it pushes them.
   A takeover entry naming another machine is D4's hand-over.
4. For a run another machine owns, unless it is `landing` past its parent-side merge, it ends this machine's processes for the slug (D9),
   archives the local tips as D4 does, and removes the local pair only, as fabric's removal under an owned-elsewhere verdict does:
   origin's branches and the board claim stay.
5. It writes this machine's pause record entries from the run pause intents of the runs it owns,
   and drops the caches of the runs it touched; each run's first pass after it is a full pass, as the dobby already owes after a sleep.

The dobby's periodic Bolt pull for the owner (`internal/dobbyengine/handover.go`) stays; a hold it reads applies at once (D7).

Bound: a catch-up that cannot reach origin acts on nothing and retries on D2's backoff; `lym beam status` shows it waiting on origin.

Rationale: the operator requires lym to check pushed state before acting after the machine slept, hibernated or was off.
The dobby already detects a sleep (`internal/dobbyengine/loop.go`), but only owes full passes and pulls Bolt up to 5 minutes later.

### D7 — Adopt holds live in Bolt; owner and adopt pause entries retire

- `board.json` gains a top-level `holds` list.
  A hold has a kind, `adopt` or `adopt-fix`, the machine, the time, and for `adopt-fix` the fix task's slug.
  Only `boardengine`'s setter writes it, pushing synchronously.
- A hold covers the whole hub: while one stands, the act rule answers `wait` for every run on every machine.
  Under `adopt-fix`, the hold's fix task alone gets `act`.
- A machine that reads a hold writes a hub entry in its own pause record, so the network gate and the turn interruption hold its runs' processes.
  It removes that entry once it reads no hold.
- A run owned by another machine is not paused: the act rule answers `hand_over`, and D9 ends its processes.
  So the `pauserecord` callers `owner`, `adopt` and legacy `adopt-hold` retire, with every writer of them:
  the dobby's owner pause, the hibernate wake's hold of runs owned elsewhere, tender's pause of runs owned elsewhere, and adopt's pause step.
  Each becomes the act rule's `hand_over` with D9's ends, keeping today's skip of a Finalize past its parent-side merge.
- At the first read after the upgrade, a standing `adopt` or `adopt-hold` entry of a run this machine owns becomes a run pause intent (D3), so no run resumes unasked.
  Every other standing entry of those callers is dropped.
- The readers of the retired adopt record that ask whether an adopt here covers a worktree, the dobby's handover and the orch watcher among them,
  read a hold this machine wrote.

Bound: a machine other than the adopter learns of a hold at its next Bolt pull, at most the dobby's pull interval, or at its next catch-up;
until then its step in flight runs to its transition.
A hold stands until the adopt that wrote it clears it or `lym adopt --release` (D8) clears it.

Rationale: the operator decided nothing happens in the hub until adopt is done and its failing tests are fixed;
the pause record is machine-local, so no other machine sees a hold kept there (state map 3.2, 6.4 items 7 and 11).

### D8 — `lym adopt` runs in the foreground: hold, owner, pull, takeover, check

`lym adopt <subject>...` runs every step in the caller's process and prints each step as it ends.
It starts no detached worker and writes no adopt record.
Each step reads tracked state, so a re-run after an interruption goes on from where the state stands.

1. One Bolt commit writes an `adopt` hold, this machine as the subjects' owner, and this machine's registry entry, and pushes it.
   A hold another machine wrote refuses the adopt, naming that machine.
   A rejected push reads Bolt again; an owner or hold of these subjects another machine wrote since the read refuses the adopt naming it;
   otherwise it replays and pushes.
2. For each moved run, it fetches the run's branches, archives this machine's local tips that origin lacks as D4 does, resets them to origin's,
   and adds the pair from origin when this machine has none.
3. It writes the takeover entry (D4) on each moved run's records branches and pushes it.
4. The catch-up (D6) over the whole hub.
5. The deploy, as today.
6. The caches it rebuilds: the hub workspace file and the orch's transcript (D10).
7. The check: the hub's `adopt` gate runs in each subject's worktree, in the foreground, with its output on the terminal.
8. A green check clears the hold and runs `orchcli.BeamUp` for each subject; the runs go on by the act rule, with no hand lift.
   A failing check files a fix task through the existing fix loop (`internal/adoptcli/fixloop.go`, `fronttask.File`),
   replaces the hold with an `adopt-fix` hold naming it, runs `orchcli.BeamUp`, and exits non-zero naming the task.
   The task's landing runs the check again; green clears the hold, and failing files the next task within `adopt_fix_tries`.
   At the limit the hold stays and the dobby's notice names the failing tests and the fix task.

`lym adopt --release` clears a hold over a failing check and leaves the fix task standing.
A shell-guard row denies it to every agent, and no refusal or notice an agent reads names it;
adopt's own terminal output at the limit and its help name it.

Kept: `--here` (behind `lym s`, now in the foreground), `--archives`, `--no-orch` and `--all`.
Retired: `--keep-local`, `--drop-local`, `--discard`, `lym adopt worker`, the adopt record, the held-back project verdicts, and the adopt pause step.

Bound: adopt works from pushed state alone and never asks about another machine's local state;
the old owner's unpushed work reaches the adopter only through D4's archive tags, which `--archives` lists.
While a hold stands the orch is up, but every verb the act rule guards refuses except on the fix task.
`--release` lets the hub go on with failing tests; only the operator's own shell runs it.

Rationale: the operator decided adopt runs in the foreground and that nothing happens in the hub until it is done and every failing test it found is fixed.
D2 to D4 leave no local state for adopt to reconcile, which removes the stops A2, A3, A5, A12 and A13.

### D9 — A moved subject's processes end, and the orch resumes its own session

- When this machine reads a subject owned elsewhere, through the dobby's handover, the orch watcher, a catch-up or D4,
  it ends every process it runs for the subject: run strands, step loops, the orch strand with its Claude process, and the dobby last.
  A run `landing` past its parent-side merge keeps its processes and finishes here.
- The orch's Claude process is registered in `internal/runproc` at launch, under an orch role keyed by the worktree;
  the registration outlives the strand's removal and is pruned when the process dies.
  Ending the subject's processes ends it through `internal/runproc`.
- At an orch start whose resume `claudeengine.Claude.CheckResume` refuses for a live holder,
  lym ends the holder when `internal/runproc` registers it as this worktree's orch, then resumes the same session id.
- A holder lym has not registered is refused, naming its pid and the way forward: exit that session, then `lym orch start`.
  The orch does not launch fresh over it.
- The orch launches fresh, with the carry-over of `internal/shuttleengine/carryover.go`, only when its transcript is neither here nor on origin (D10),
  and the start says so.

Rationale: on both recorded adopts the orch's own earlier session, still alive on the adopting machine after its strand was removed, held the transcript,
and `internal/orchcli/start.go` fell back to a fresh launch on any resume error (A6, A7).

### D10 — The orch's transcript travels through origin

- The orch's session id moves from the machine-local orch state (`orchengine.State`) to a tracked record under the worktree's `_lym/orch/`.
  The orch start commits and pushes it at each launch and resume, as D2's decision writes do.
- The orch's transcript is mirrored to the weft repository under one ref per worktree, outside every branch, as a single commit each mirror replaces.
  The commit names the machine that wrote it.
  The transcript is split into chunks below GitHub's file size limit, in order and uncompressed, so a mirror uploads only the changed last chunk as a delta.
- A mirror runs at the orch's turn end at most once per mirror interval, and at each orch stop and hibernate request,
  only while the act rule does not answer `hand_over` for the worktree's subject.
  It pushes with a lease on the ref's last value this machine read; a lost lease ends the mirror and writes nothing.
  It runs detached and best effort, logs its outcome, and holds no run; one killed mid-push leaves the ref as it was.
- At an orch start, when origin's mirror names another machine, or this machine has no transcript, lym restores the mirror
  into the Claude project directory of this worktree's path, keeping a local transcript it replaces beside it under another name, then resumes by id.
  When the mirror names this machine, the local transcript wins.
- The mirror, the restore and the transcript's location live in `internal/shuttleengine/claudeengine`.

Bound: a crash loses on another machine the orch's turns since the last mirror, at most one mirror interval.
Only the orch's transcript travels; a run's agent sessions start their step again (D3).

Facts this rests on:

- Claude Code keeps a session's transcript at `~/.claude/projects/<cwd with non-alphanumerics as "-">/<id>.jsonl` (state map 3.3),
  and resumes from the cwd's project directory: lym's resume check stats the transcript there (`internal/shuttleengine/claudeengine/resume.go`),
  and every orch resume today passes that check before Claude Code resumes it.
- `claude --resume <id>` keeps the session id; `--fork-session` is the flag that makes a new one (`claude --help`, Claude Code 2.1.325, read 2026-11-15).
- The prime's orch transcripts on `neve` reach 206 MB for one session (measured 2026-11-15), and GitHub refuses a file over 100 MB.

Rationale: the operator wants the orch back as a continuation, never as a new session, on whichever machine it goes on.
A ref replaced on each mirror keeps one copy on origin and no history to grow.
Mirroring only while owner, under a lease, means the copy on origin is the latest owner's.

### D11 — Leftover pairs are torn down or reported

- The dobby tears down, through `internal/pairteardown`, the pair of each run in `landed` or `cancelled` without keep.
  This widens the ended-run teardown of `internal/dobbyengine/ended.go`, whose conditions read the FSM state and the act rule in place of the cancel marker and the pause record.
- A cancelled run whose pair was kept stays: `lym beam status` lists it with its age,
  and each dobby start posts one notice listing kept pairs and naming `lym fabric remove <slug>`.
- A pair with neither a board entry nor a batten record, such as a bench's or a trial's, is an orphan.
  The dobby posts a notice at first sight saying when it goes, and tears it down no earlier than the orphan grace after that notice,
  and only when neither branch has had a commit for the orphan grace and no process of its slug lives.
  Uncommitted files are committed onto the archived tip first.
- `lym tender resume` pushes no pair, since D2 leaves no deferred push for it to pay; the `rerun` pair's failure on every resume ends.
- The hub workspace file is regenerated after each teardown, so a torn-down pair leaves VS Code.

Bound: an orphan in use stays while it commits or runs a process; an idle one goes after the grace, its branches kept under archive tags.
A kept pair's batten record keeps it out of the orphans, also when its board entry is removed.

Rationale: a bench pair, a kept pair and a trial pair stood for days, listed in the operator's VS Code, with nothing reporting them (A14).
A kept pair stays because keeping it was the operator's explicit choice at the cancel.

## Constraints

- `PATTERN-owner-machine`: the owner stays on roots and beams, written only by `boardengine`'s setter, read through `hubgeom.ParentOf`.
  The rest of a Finalize past its parent-side merge still finishes on whichever machine runs it (D1, D6, D9).
- `PATTERN-pair-remote`: the code branch still goes along with a pair's records push, and a refused push still leaves a failed-push record and a notice.
  A catch-up's local pair removal (D6) deletes no origin branch.
- `PATTERN-dobby-runs`: runs are still started only by the parent's dobby, which starts only with its orch bar `--no-orch`.
- `PATTERN-lym-files`: every new tracked file sits under `_lym` and every cache under `.lym`, both named in `internal/lymdirs`.
- `PATTERN-tolerant-records`: the new tracked records and the Bolt `machines` and `holds` keys read tolerantly and are written through `internal/state` or the board store.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the synchronous push, the takeover entry's push, the archive and the transcript ref run in `internal/fabricengine` through `gitexec` and `gitrepo`.
- `PATTERN-shuttle-provider-seam`: Claude's transcript paths and the mirror's content handling live in `internal/shuttleengine/claudeengine`.
- `PATTERN-child-processes`: the mirror is a logged, registered detached child; the orch's Claude process registers in `internal/runproc` (D9), and D9's ends go through it.
- `PATTERN-daemon-wakeups`: D2's and D6's retry waits wake at most once a second and back off while idle.
- `PATTERN-refusal-way-forward`: every refusal the act rule, the holds and D9 add names its way forward and a test reaches it;
  none an agent reads names `lym adopt --release`, which agents' settings deny (D8).
- `PATTERN-shell-guard-table`: the deny of `lym adopt --release` is a row of `internal/shellguard`'s table.
- `PATTERN-code-conventions`: the mirror interval, the orphan grace and the chunk size are each one named constant.

## Auto-mode assumptions

- The mirror interval is 10 minutes (D10) and the orphan grace 24 hours (D11).
- The other picks the question ledger records stand as D1 to D11 state them.

## Open risks

- D10 pushes the orch's conversation to the weft repository on GitHub. The parent decides whether that repository's visibility and the transcripts' contents allow it.
- D2 stops every run while origin is unreachable. The parent confirms the operator accepts no offline progress.
- D8's `lym adopt --release` lets the hub go on with failing tests, against the operator's "until every failing test it found is fixed".
  The parent confirms the operator wants that escape for a check that cannot pass on this machine.
- D8 brings the orch up while a hold stands, with every run verb refused bar the fix task's; the entry lists "no orch action" beside "no run starts".
  The parent confirms that reading.
- `claude-account-pick-simple` (T1, draft) makes a worktree's Claude config directory per machine; D10's restore must target the directory that design resolves.

## Acceptance criteria

- D1: for each state, a fixture of tracked records alone reads that state through `internal/runfsm` (new), with every `.lym` cache absent; a fixture meeting two states' conditions reads the one earlier in the order.
- D1: the act rule answers `hand_over` for a run another machine owns, `act` for one `landing` past its parent-side merge there, `wait` under a hold, a pause or an owed push with origin unreachable, and `act` otherwise.
- D1: the dobby's pass, a hand `lym batten run`, `lym loom resume` and `lym tender resume` each refuse a run the act rule answers `wait` or `hand_over` for, naming the way forward.
- D2: a shed transition in a task pair starts the next producer only after both sides' push landed on origin.
- D2: with origin unreachable, a transition leaves the run waiting with the push owed, starts no producer, and goes on once origin answers.
- D2: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns.
- D3: after `lym batten cancel` returns, the cancel intent is on origin in the run's batten record; a cancel interrupted on one machine reads `cancelling` on another.
- D3: `lym tender pause <slug>` puts the pause intent on origin and holds the run's processes here; a pause of a run another machine owns is refused naming that machine.
- D3: a pull-request record and the fixer attempt counts written on one machine are read on another after a pull.
- D3: a run whose `.lym` is removed reads the same FSM state as before the removal.
- D4: a records push rejected over a takeover entry naming another machine ends this machine's processes for the slug, pushes an `archive-owner/` tag holding its local commits and uncommitted files, and pushes nothing to the branch.
- D4: a push rejected over another run's commits on a shared records branch replays and pushes, and the run goes on.
- D4: an adopter's takeover push rejected over the old owner's later transition replays and lands, and the adopter reads the run at that transition.
- D5: an owner recorded as a former name of a registered machine reads as owned here on that machine and as that machine on another.
- D5: a machine's first registration carries the former names of its `machine.json`; `lym machine name <new>` puts the old name among them on origin when it returns.
- D6: a dobby that observes a sleep runs the catch-up before its next pass; a run another machine took meanwhile gets no pass, its processes end, and its local pair is removed with its origin branches left standing.
- D6: a run owned here whose local records are ahead of origin with its own commits has them pushed by the catch-up, and none is reset.
- D6: a run owned elsewhere at `Finalize` past its parent-side merge keeps its processes and pair through the catch-up.
- D6: with origin unreachable, the catch-up acts on no run and retries.
- D7: under an `adopt` hold no queued task is claimed and no run gets a pass, on the adopting machine and on another machine after its Bolt pull; under `adopt-fix` the fix task alone gets passes.
- D7: a machine reading a hold holds its runs' processes through its pause record, and lets them go once the hold clears.
- D7: a standing `adopt` pause entry from before the upgrade becomes a pause intent on its run, which stays paused; a standing `owner` entry is dropped, and no code path writes either caller after the upgrade.
- D8: `lym adopt <beam>` from a second machine's hub, with origin as the only shared state, ends with the beam's runs in the same FSM states as on the first machine and this machine as owner, with no `--keep-local` or `--drop-local`.
- D8: an adopt run on a machine with no local pair of a moved run adds the pair from origin before it writes the takeover entry.
- D8: two machines adopting at once end with one owner; the other's adopt refuses naming the winner.
- D8: a failing adopt check files one fix task, leaves an `adopt-fix` hold naming it, and exits non-zero; the task's green re-check clears the hold and the runs go on with no hand lift.
- D8: `lym adopt --release` clears a hold with the fix task standing; an agent's shell is denied it.
- D8: an adopt killed after its owner write goes on from the tracked state on its re-run and ends as an uninterrupted adopt does.
- D9: an orch start whose session is held by this worktree's own earlier orch process, its strand already removed, ends that process and resumes the same session id.
- D9: an orch start whose session is held by a process lym has not registered refuses, naming the pid, and launches nothing fresh.
- D9: a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby running on this machine.
- D10: the transcript mirror of a fake transcript larger than the chunk size round-trips byte for byte through a fake origin, and a second mirror after an append uploads only the last chunk.
- D10: an orch start on a second machine with no local transcript restores origin's mirror into the project directory of its own worktree path and launches with `--resume` and the same session id.
- D10: an orch start whose local transcript is longer than a mirror another machine wrote restores the mirror and keeps the local copy beside it.
- D10: a mirror from a machine the act rule answers `hand_over` for writes nothing, and one whose lease is lost writes nothing.
- D11: a cancelled run's pair without keep is torn down by the dobby; a kept pair stays and is listed with its age in `lym beam status`, also after its board entry is removed.
- D11: an orphan with no commit and no live process is torn down no earlier than the orphan grace after its notice, its branches archived; one with a commit inside the grace stays.
- D11: `lym tender resume` pushes no pair.

## PATTERN changes

- `PATTERN-run-fsm` — adds: Reading or acting on a run's lifecycle state: only `internal/runfsm` (new)'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until its check is green or `lym adopt --release`.
- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along, synchronously before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.

## Notes for the plan writer

- A split that raddle can follow, in dependency order:
  `internal/runfsm` (new) and its readers (D1); tracked records and caches (D3); synchronous push and the takeover entry (D2, D4);
  the registry (D5); the holds and the retired pause callers (D7); the catch-up (D6); adopt (D8); orch end, resume and transcript (D9, D10); leftover pairs (D11).
- The two-machine acceptance criteria need two hubs over one fake origin with different machine names; `hubforge` builds hub fixtures.
- `internal/adoptcli`'s package doc describes today's worker, record and verdicts at length; most of it retires with D8.
