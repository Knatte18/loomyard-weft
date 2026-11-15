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

1. `landed`: the batten record `done`, or no batten record and the board status `done`, as the dobby's done-run clear leaves a landed run.
2. `cancelled`: a cancel intent with its finish (D3), the finish saying whether the pair was kept.
3. `cancelling`: a cancel intent (D3) without a finish.
4. `paused`: a run-scope pause intent (D3), or a shed state `paused`.
5. `halted`: the child run `blocked`, `failed` or `awaiting`.
6. `landing`: the child run at `Finalize` or a later row.
7. `running`: the batten record at `Run-Shed` or later.
8. `starting`: a batten record at a row before `Run-Shed`.
9. `queued`: a released task with an empty board status and no batten record.

A pair stands or not beside each state; a run in the states D11 names whose pair stands owes a teardown (D11).

The act rule answers, for one run on this machine, `act`, `wait` or `hand_over`.
Its inputs are the run's state, the owner read through `hubgeom.ParentOf` and the registry (D5), the Bolt holds (D7),
whether origin's code branch holds the code tip the run's last transition names (D4),
and two machine-local facts: a push owed with origin unreachable (D2), and this machine's pause record coverage.

- `wait` when a push is owed and origin is unreachable.
- `hand_over` when another machine owns the run; its owner finishes even a `landing` past its parent-side merge (D4).
- `wait` when origin's code branch lacks the code tip the run's last transition names (D4).
- `act` when the run is `landing` past its parent-side merge and no pause covers it, so it finishes under a hold.
- `wait` when a hold covers it, a pause covers it, the state is `paused` or `cancelling`, or the state is terminal.
- `act` otherwise.

Every verb or loop that acts on a run asks the act rule first, and every reader of a run's state takes it from `internal/runfsm` (new).
`lym tender resume <slug>` and `lym batten cancel` set the run's own pause and cancel intent aside when they ask,
and `lym batten cancel` sets a hold aside too, so the operator can resume or cancel a held run.
The board's `<state> · <row>` status is a display written from the FSM state; no decision reads it but `queued`'s empty status and `landed`'s `done` with no batten record.

Rationale: the state map (section 6.4) finds six vocabularies that disagree on `paused`, `running`, `done` and the halted states.
One package over tracked records gives every machine the same state from the same pushed bytes.

### D2 — A transition pushes its records before the run acts

- A shed transition commits its records as today, the files D3 moves into tracked records included, then pushes them synchronously:
  in a task pair the records side first, and the code side only once the records side has landed;
  records only in the prime.
  The records commit names the code tip the transition was made with (D4).
  The next producer starts only once every side has landed.
- Every detached status push (`fabricengine.SpawnStatusPush`) that carries a run transition retires:
  the per-transition spawn behind `internal/statuscommit/statuscommit.go`,
  and the respawn of a pair's push that exited on a held teardown lock (`internal/fabricengine/teardownlock.go`).
  A transition's push that meets a held teardown lock waits for its release and pushes before the next producer starts.
  Fabric's pause deferral of a push no longer applies to a transition's push, which is synchronous;
  it stays for the detached pushes below.
  A transition takes a network gate admission (`pauserecord.Admit`) before its commit,
  and releases it once every side has landed or its push turns owed,
  so a pause or hold written after the commit stops the run's next producer and never that push's first attempt.
  An owed push's retries wait at the gate like any network call, so a pause holds them, and the run waits on the owed push meanwhile.
  `lym loom commit-records` keeps its synchronous push.
- The anchor change's push (`internal/fabricengine/anchorchange.go`) and tender's prime push (`internal/tendercli/deps.go`) stay detached.
  Neither carries a run transition: the first pushes a pair's anchor move,
  and the second pushes only commits no run waits on, since every transition pushed before its run acted.
- A push that fails for a transient reason, such as an unreachable origin, leaves the run waiting at that transition with the push owed.
  It retries on a backoff under `PATTERN-daemon-wakeups` and starts no producer meanwhile.
  `lym beam status` shows the owed push and its last error, and the existing failed-push record and creel notice stay.
- A rejected push reads origin's tip for a takeover first (D4), and replays the records as today only when it finds none.
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
  `lym batten cancel` commits and pushes it before it ends anything, or acts offline as below.
  A cancel keeps the batten record, in place of today's removal of the run's records,
  and writes its finish into it, the time it ended and remove or keep, committed and pushed when the cancel ends.
  The finish sets the shed state `cancelled`, never `done`,
  so the dobby's done-run clear (`battencli.ClearDoneRun`), which deletes the task branch on origin, never takes a cancelled run.
  `lym batten cleanup` removes a `cancelled` record whose pair is gone.
  It refuses one whose pair stands, naming `lym fabric remove <slug>` as the way forward.
  A cancelled run's batten record goes with the pair's teardown, D11's or `lym fabric remove <slug>`;
  a slug released again then reads `queued`.
  A landed run's record goes at the dobby's done-run clear as today, and the run reads `landed` from its board `done` (D1).
- A run-scope pause intent, in the batten record: who paused and when.
  `lym tender pause <slug>` writes it and `lym tender resume <slug>` clears it, each committing and pushing before it returns.
  Each also writes this machine's pause record entry as today, which the network gate and the turn interruption read.
- The pull-request record of `internal/pullrequest/record.go`, whole, in the pair's records; a write of it commits and pushes at once.
- The fixer attempt counts of `internal/landingshed/fixerattempts.go`, in the pair's records, committed with the transition.

A pause or cancel of a run another machine owns is refused, naming that machine, so only the owner writes a run's records.
Its way forward is the same verb on that machine, or `lym adopt` of the run's subject here first.
The verb reads the owner from a Bolt it fetches first.
A machine that becomes a run's owner writes its pause record entry from the run's pause intent at its catch-up (D6).

With origin unreachable, each of these verbs acts here at once and owes its push under D2's retry:

- `lym tender pause <slug>` writes this machine's pause record entry, so the run's processes stop, and commits the intent.
- `lym tender resume <slug>` commits the cleared intent; the run waits on the owed push (D1) until it lands.
- `lym batten cancel` ends the run's processes and strands and commits the intent.
  Its output names as way forward a re-run of `lym batten cancel <slug>` once origin answers,
  which pushes the intent and finishes the cancel, its finish and any pair removal.

Offline, the owner refusal reads the local Bolt, and a takeover the owed push meets is D4's hand-over.
Bound: until the owed push lands, the intent is on this machine only, and another machine reads the run unpaused or not cancelled.
When the owed push meets a takeover, the hand-over drops the intent, and its notice names each pause or cancel intent it dropped,
to repeat on the owner machine.

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
- When a run's records push is rejected, lym fetches the branch and reads the run's own record from origin's fetched tip, before any replay.
  When that record holds a takeover entry naming another machine that this machine has not yet read, the run hands over without replaying:
  this machine pushes no code side for the rejected transition, ends its processes for the slug (D9),
  pushes the code tip the run's last transition on origin names when origin's code branch lacks it, since that transition stands,
  archives the local tips as the old owner's archive does (`fabricengine.ArchiveOwnerTips`), with uncommitted files first committed onto the archived tip,
  pushes the archive tags, and posts one notice naming the machine and any pause or cancel intent it dropped (D3).
  It pushes nothing more for the run, so its code commits after the takeover reach origin only under the archive tags.
  A run `landing` past its parent-side merge hands over too, and its new owner resumes the Finalize from origin's records.
  Its parent-side merge is a transition, on origin before the Finalize goes on (D2),
  and each later push of the Finalize first reads the owner from a fetched Bolt and hands over when another machine owns the run,
  so the old owner writes nothing to the parent's branch after it reads the takeover.
- Each transition's records commit names the code tip it was made with (D2).
  After a takeover, origin's code branch therefore moves only to a tip a pushed transition names,
  so the adopter's later code push meets no other machine's commit.
  Until origin's code branch holds the tip the run's last transition names, the act rule answers `wait` (D1),
  and `lym beam status` names the tip and the machine that owes it;
  that machine's next hand-over or catch-up (D6) pushes it,
  and the owner's dobby fast-forwards its local code branch to it before the run's next pass.
  A tip still missing after the code-tip grace is given up:
  the owner commits and pushes a records commit that returns the run to its previous transition,
  and the step runs again from there, as D2's bound says a stopped step does.
  A machine owing a tip pushes it only while the run's last transition on origin names it, so a given-up tip never lands.
- A rejection whose fetched record holds no such entry, such as one over commits of other runs on a shared records branch like `main-weft`,
  replays and pushes as today.
- An adopter's own takeover push that is rejected fetches, then re-reads its hold and the subjects' owner from a Bolt it fetches.
  When its hold is gone or either names another machine, it stops, naming the release (D8).
  Otherwise it writes the takeover entry again onto origin's tip and pushes again;
  it never replays its own commit over a record the old owner changed, and it never hands over.

Bound: transitions the old owner pushed before the takeover landed stand, and the adopter goes on from them.
The old owner's step in flight runs to its next transition, which the takeover rejects;
its result is archived under a tag, not lost and on neither branch.

Rationale: a takeover entry in the run's own record is a compare-and-swap per run that holds on records branches many runs share,
and needs no extra fetch per transition.

### D5 — A machine registry in Bolt gives each machine one identity

- `board.json` gains a top-level `machines` map: each registered name with the former names it replaced.
  Only `boardengine`'s owner setter writes it, pushing synchronously.
- A machine registers at its first owner write or `lym machine name`, copying the former names of its `machine.json`.
  A registration whose former name is another registered machine's current name is refused, naming that machine;
  its way forward is `lym machine name --forget-former <name>` on this machine, which drops that former name from its `machine.json`.
- `lym machine name` records the rename there, besides its sweep of owners (`internal/machinecli/set.go`).
- `lym machine name --forget-former <name>` (new) drops a former name from this machine's `machine.json` and its registry entry;
  owners recorded under that name then read as the machine whose current name it is.
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
4. For a run another machine owns, a `landing` past its parent-side merge included, it ends this machine's processes for the slug (D9),
   pushes the code tip the run's last transition on origin names when origin lacks it (D4), and archives the local tips as D4 does.
   It then removes the local pair through `internal/pairteardown`'s local mode (`Request.Local`),
   which takes no origin step and writes no board: origin's branches and the board claim stay.
5. It writes this machine's pause record entries from the run pause intents of the runs it owns,
   and drops the caches of the runs it touched; each run's first pass after it is a full pass, as the dobby already owes after a sleep.

The dobby's periodic Bolt pull for the owner (`internal/dobbyengine/handover.go`) stays; a hold it reads applies at once (D7).

Bound: a catch-up that cannot reach origin acts on nothing and retries on D2's backoff; `lym beam status` shows it waiting on origin.

Rationale: the operator requires lym to check pushed state before acting after the machine slept, hibernated or was off.
The dobby already detects a sleep (`internal/dobbyengine/loop.go`), but only owes full passes and pulls Bolt up to 5 minutes later.

### D7 — Adopt holds live in Bolt; owner and adopt pause entries retire

- `board.json` gains a top-level `holds` list.
  A hold has a kind, `adopt` or `adopt-fix`, the machine, the time, and for `adopt-fix` the fix task's slug and each subject's try count (D8).
  Only `boardengine`'s setter writes it, pushing synchronously.
- A hold covers the whole hub: while one stands, the act rule answers `wait` for every run on every machine.
  Under `adopt-fix`, the hold's fix task gets `act`, and a run `landing` past its parent-side merge is not held under either kind.
- Every verb that asks the act rule outside the dobby's pass, such as `lym loom resume` and `lym tender resume <slug>`,
  fetches Bolt first when origin answers, and reads the local Bolt only when it does not.
- A machine that reads a hold writes no hub entry.
  It writes a slug entry in its own pause record, under one new caller `hold`,
  for each run of this machine the act rule answers `wait` for under the hold, and none for the `adopt-fix` hold's fix task,
  so the network gate and the turn interruption hold those runs' processes.
  It drops those entries once it reads no hold.
- The `hold` entries hold nothing outside those runs.
  The orch, `lym adopt`'s own process and the children it spawns for the deploy and the check,
  and the operator's `lym batten cancel`, which today only a hub entry refuses, go on under a hold.
- The `hold` entries keep apart from the operator's hub entry and hibernate's.
  Dropping them drops only entries of the `hold` caller,
  and the hibernate wake, which skips its reconcile under a hub entry, never reads a `hold` slug entry as one.
  A slug that already has a live entry gets no `hold` entry, since the pause record keeps one live entry per scope.
  `lym tender resume` lifts no `hold` entry while the Bolt hold stands;
  when it lifts a slug's own entry under a standing hold, it writes the `hold` entry in its place.
  `lym tender resume <slug>` of a slug only a `hold` entry covers refuses, naming the hold's machine,
  and as way forward waiting for the hold to clear, or asking the operator when that machine has stopped.
- A run owned by another machine is not paused: the act rule answers `hand_over`, and D9 ends its processes.
  So the `pauserecord` callers `owner`, `adopt` and legacy `adopt-hold` retire, with every writer of them:
  the dobby's owner pause, the hibernate wake's hold of runs owned elsewhere, tender's pause of runs owned elsewhere, and adopt's pause step.
  Each becomes the act rule's `hand_over` with D9's ends; a Finalize past its parent-side merge hands over too, and its owner finishes it (D4).
- At the first read after the upgrade, a standing `adopt` or `adopt-hold` entry of a run this machine owns in a state that is not terminal
  becomes a run pause intent (D3), so no run resumes unasked.
  Every other standing entry of those callers is dropped.
  A standing `adopt` or `adopt-hold` entry also makes that first read write one Bolt `adopt` hold naming this machine, so no queued task starts;
  a re-run of `lym adopt` here clears it at a green check, as does `lym adopt --release`.
  That first read posts one notice to the operator naming the hold, its machine,
  and as the way forward a re-run of `lym adopt` on that machine of the subjects those entries cover.
  A first read that finds another machine's hold standing writes none, so one machine holds the hub at a time;
  its notice names that hold's machine, and the re-run here once that hold clears.
  Like every notice an agent reads, neither names `--release`.
  A released task's entry with no batten record is dropped, and the task stays `queued` under that hold.
- The readers of the retired adopt record that ask whether an adopt here covers a worktree, the dobby's handover and the orch watcher among them,
  read a hold this machine wrote.

Bound: a machine other than the adopter learns of a hold at its next Bolt pull or catch-up:
its dobby's pull within the dobby's pull interval, and a verb's before it asks the act rule.
Until then its dobby goes on with its runs as before, for at most the dobby's Bolt pull interval; D4's bound covers the transitions it pushes.
A verb that cannot reach origin reads the local Bolt, and any transition it leads to waits on origin under D2.
A run that holds its parent lock is exempt from slug entries, so the gate does not hold its step in flight;
the act rule still starts no producer for it.
A hold stands until the adopt that wrote it clears it, or `lym adopt --release` (D8) clears it on any machine;
a hold no live adopt owns, such as one whose adopter stopped, stands until the operator releases it.

Rationale: the operator decided nothing happens in the hub until adopt is done and its failing tests are fixed;
the pause record is machine-local, so no other machine sees a hold kept there (state map 3.2, 6.4 items 7 and 11).

### D8 — `lym adopt` runs in the foreground: hold, owner, pull, takeover, check

`lym adopt <subject>...` runs every step in the caller's process and prints each step as it ends.
It starts no detached worker and writes no adopt record.
Each step reads tracked state, so a re-run after an interruption goes on from where the state stands.

1. One Bolt commit writes an `adopt` hold, this machine as the subjects' owner, and this machine's registry entry, and pushes it.
   A hold another machine wrote refuses the adopt, naming the writer machine and the hold's time,
   and as way forward waiting for that machine's adopt to clear it, or asking the operator when that machine has stopped.
   A rejected push reads Bolt again; an owner of these subjects, or any hold, that another machine wrote since the read
   refuses the adopt naming that machine; otherwise it replays and pushes.
   A registration D5 refuses refuses the adopt with D5's way forward.
2. For each moved run, it fetches the run's branches, archives this machine's local tips that origin lacks as D4 does, resets them to origin's,
   and adds the pair from origin when this machine has none.
3. It writes the takeover entry (D4) on each moved run's records branches and pushes it.
   Once it lands, it fast-forwards each moved run's local records branches to origin's tips, which the takeover entry now heads,
   and its local code branch to origin's code tip.
   A code tip the last transition names that origin still lacks is the owner's dobby's to fast-forward to once it lands (D4).
4. The catch-up (D6) over the whole hub.
5. The deploy, as today.
6. The caches it rebuilds: the hub workspace file and the orch's transcript (D10).
7. The check: the hub's `adopt` gate runs in each subject's worktree, in the foreground, with its output on the terminal.
8. A green check clears the hold and runs `orchcli.BeamUp` for each subject; the runs go on by the act rule, with no hand lift.
   A failing check files a fix task through the existing fix loop (`internal/adoptcli/fixloop.go`, `fronttask.File`),
   replaces the hold with an `adopt-fix` hold naming it, runs `orchcli.BeamUp`, and exits non-zero naming the task.
   The task's landing runs the check again; green clears the hold, and failing files the next task within `adopt_fix_tries`.
   Each subject's try count lives in the `adopt-fix` hold, rewritten with each filing,
   so a re-check on any machine reads the same count against that subject's limit.
   At the limit the hold stays and the dobby's notice names the failing tests and the fix task.

`lym adopt --release` clears any standing hold, `adopt` or `adopt-fix`, whichever machine wrote it, and leaves a fix task standing.
An adopt re-reads its own hold from Bolt before each step from step 2 on, and stops, naming the release, once the hold is gone,
so a release over a live adopt leaves one adopter.
An adopt that stops on a release after its step 3 landed still runs `orchcli.BeamUp` for the subjects it owns, so their runs go on by the act rule;
one that stops earlier names a re-run of `lym adopt` as its way forward, since its moved runs have no takeover entry yet.
A shell-guard row denies it to every agent, and no refusal or notice an agent reads names it;
adopt's own terminal output at the limit and its help name it.

Kept: `--here` (behind `lym s`, now in the foreground), `--archives`, `--no-orch` and `--all`.
Retired: `--keep-local`, `--drop-local`, `--discard`, `lym adopt worker`, the adopt record, the held-back project verdicts, and the adopt pause step.

Bound: adopt works from pushed state alone and never asks about another machine's local state;
the old owner's unpushed work reaches the adopter only through D4's archive tags, which `--archives` lists,
besides the code tip of a transition already on origin (D4).
While a hold stands the orch is up, but every verb the act rule guards refuses except on the fix task and a landing past its parent-side merge.
The hold's pause entries cover only held runs (D7),
so the orch, adopt's deploy and check with their children, and the operator's `lym batten cancel` go on.
`--release` lets the hub go on with failing tests, or without the check of an adopter that stopped; only the operator's own shell runs it.

Rationale: the operator decided adopt runs in the foreground and that nothing happens in the hub until it is done and every failing test it found is fixed.
D2 to D4 leave no local state for adopt to reconcile, which removes the stops A2, A3, A5, A12 and A13.

### D9 — A moved subject's processes end, and the orch resumes its own session

- When this machine reads a subject owned elsewhere, through the dobby's handover, the orch watcher, a catch-up or D4,
  it ends every process it runs for the subject: run strands, step loops, the orch strand with its Claude process, and the dobby last.
  A run `landing` past its parent-side merge ends here too, and its owner finishes it (D4).
- The orch's Claude process is registered in `internal/runproc` at launch, under an orch role keyed by the worktree;
  the registration outlives the strand's removal and is pruned when the process dies.
  Ending the subject's processes ends it through `internal/runproc`.
- At an orch start whose resume `claudeengine.Claude.CheckResume` refuses for a live holder,
  lym ends the holder when `internal/runproc` registers it as this worktree's orch, then resumes the same session id.
- A holder lym has not registered is refused, naming its pid and the way forward: exit that session, then `lym orch start`.
  The orch does not launch fresh over it.
- An orch Claude process the pre-upgrade lym launched is unregistered too,
  so the first orch start after the upgrade refuses over it, naming its pid, and the operator exits that session once.
- The orch launches fresh, with the carry-over of `internal/shuttleengine/carryover.go`, only when its transcript is neither here nor on origin (D10),
  and the start says so.

Rationale: on both recorded adopts the orch's own earlier session, still alive on the adopting machine after its strand was removed, held the transcript,
and `internal/orchcli/start.go` fell back to a fresh launch on any resume error (A6, A7).

### D10 — The orch's transcript travels through origin

- The orch's session id moves from the machine-local orch state (`orchengine.State`) to a tracked record under the worktree's `_lym/orch/`.
  The record also names the machine that last launched or resumed the orch.
  An orch start writes it only after a Bolt it fetches names this machine as the subject's owner, and starts nothing otherwise (D9).
  The orch start commits and pushes it at each launch and resume, as D2's decision writes do.
- The orch's transcript is mirrored to the weft repository under one ref per worktree, outside every branch, as a single commit each mirror replaces.
  The commit names the machine that wrote it.
  The transcript is split into chunks below GitHub's file size limit, in order and uncompressed, so a mirror uploads only the changed last chunk as a delta.
- A mirror runs at the orch's turn end at most once per mirror interval, and at each orch stop and hibernate request,
  only while the act rule does not answer `hand_over` for the worktree's subject.
  A turn end inside the interval schedules one trailing mirror at the interval's end, so the last turn reaches origin.
  It pushes with a lease on the ref's last value this machine read.
  A mirror whose lease is lost re-reads the ref and the session record on origin.
  When the ref names another machine that the session record names as the orch's last launcher or resumer,
  it stops and writes nothing;
  otherwise it pushes again with a lease on the value re-read, while the act rule still does not answer `hand_over`.
  The act rule's `hand_over` reads the Bolt owner, so the old owner's mirror stops once it reads the takeover.
  It runs detached and best effort, logs its outcome, and holds no run; one killed mid-push leaves the ref as it was.
- At an orch start, when the session record names another machine as the orch's last launcher or resumer,
  or this machine has no transcript, lym restores the mirror into the Claude project directory of this worktree's path,
  keeping a local transcript it replaces beside it under another name, then resumes by id.
  When the session record names this machine, the local transcript wins, whatever machine the mirror names.
  When no mirror stands on origin, a local transcript is resumed as it is.
- The mirror, the restore and the transcript's location live in `internal/shuttleengine/claudeengine`.

Bound: a machine that crashes, or is taken over before its trailing mirror lands,
loses on another machine the orch's turns since its last mirror, at most one mirror interval.
Only the orch's transcript travels; a run's agent sessions start their step again (D3).
After an adopt, the old owner's late mirror can land for up to the dobby's Bolt pull interval.
The new owner's start never restores it, since the session record names the new owner,
and the new owner's next mirror replaces it; the old owner's turns after the takeover do not travel.

Facts this rests on:

- The weft repository `Knatte18/loomyard-weft` is public (`gh repo view Knatte18/loomyard-weft --json visibility`, read 2026-11-15),
  so anyone can read a ref pushed to it.
- Claude Code keeps a session's transcript at `~/.claude/projects/<cwd with non-alphanumerics as "-">/<id>.jsonl` (state map 3.3),
  and resumes from the cwd's project directory: lym's resume check stats the transcript there (`internal/shuttleengine/claudeengine/resume.go`),
  and every orch resume today passes that check before Claude Code resumes it.
- `claude --resume <id>` keeps the session id; `--fork-session` is the flag that makes a new one (`claude --help`, Claude Code 2.1.325, read 2026-11-15).
- The prime's orch transcripts on `neve` reach 206 MB for one session (measured 2026-11-15), and GitHub refuses a file over 100 MB.

Rationale: the operator wants the orch back as a continuation, never as a new session, on whichever machine it goes on.
A ref replaced on each mirror keeps one copy on origin and no history to grow.
Mirroring only while owner, under a lease, means the copy on origin is the latest owner's.

### D11 — Leftover pairs are torn down or reported

- The dobby tears down, through `internal/pairteardown`, the pair of each run in the FSM state `landed`, or `cancelled` without keep,
  with no pause or hold covering it and no other machine owning it.
  This widens the ended-run teardown of `internal/dobbyengine/ended.go`,
  whose conditions read these in place of the cancel marker and the pause record.
- A cancelled run whose pair was kept stays: `lym beam status` lists it with its age,
  and each dobby start posts one notice listing kept pairs and naming `lym fabric remove <slug>`.
- A pair kept by a cancel before the upgrade has no batten record left.
  The first dobby start after the upgrade writes, for each pair whose slug has a board entry, no batten record and no live process,
  a batten record holding a finished cancel with keep, so the pair is listed as kept and never read as `queued` or as an orphan.
  The board entry must be unclaimed, as a cancel leaves it, so a pair whose claim stands is never read as kept.
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

### Added after Discussion (arbiter, 2026-11-15): No run acts while origin is unreachable

Decision: D2's bound stands as written: no run acts while origin is unreachable, and an owed push holds the run at its transition until it lands. This answers review finding F15 (no-offline-progress-unconfirmed). The writer removes the Open risks bullet 'D2 stops every run while origin is unreachable. The parent confirms the operator accepts no offline progress.' and changes nothing else in D2, D3 or D6 for it.

Rationale: The board entry's operator requirement (2026-11-15) says the whole state of lym and of every run is stored on GitHub at all times, and its Direction says each transition writes and pushes its record before it acts. A run acting offline would leave a state on this machine only, which that requirement rules out; the support log's rejected alternative 'letting a run act offline and pushing later' already says so.

### Added after Discussion (arbiter, 2026-11-15): No lym adopt --release; a hold clears only at a green check

Decision: Answers review finding F13 (release-verb-against-operator-decision): no verb lets the hub go on past an adopt hold before an adopt's check is green. The writer drops 'lym adopt --release' from D7 (the upgrade bullet and the Bound), D8 (the release paragraph, the stop-on-release lines, the Bound), the Constraints (the shell-guard row and the refusal-way-forward line), the declared PATTERN-dobby-runs change and the acceptance criteria that name it, and removes the Open risks bullet on --release. D8 instead states how a hold whose adopter stopped is cleared without the hub going on: an operator's lym adopt of the same subjects on another machine, given an explicit flag the writer names, takes over a hold another machine wrote by rewriting it to name this machine, then runs its own steps and check, so the hold clears only at that adopt's green check or, as an adopt-fix hold, at the fix task's green re-check. That flag is denied to agents by a shell-guard row and no text an agent reads names it; the stopped adopter, if it returns, stops at its next hold re-read as D8 already says. At the fix-try limit the hold stays until the failing tests are fixed and a re-run of lym adopt checks green.

Rationale: The board entry's Operator decisions (2026-11-15) say nothing else happens in the hub, no run start, no dobby pass and no orch action, until adopt has finished and every failing test it found is fixed. A release that lets the hub go on with failing tests, or without the check of an adopter that stopped, contradicts that directly, as the record's own Open risks line says. A hold taken over by a second adopt keeps the stopped-adopter case recoverable while the hub still waits for a green check.

### Added after Discussion (parent, 2026-11-15): No orch transcript mirror (F12)

Decision: Drop D10's transcript mirror to the weft repository and every cross-machine restore of the orch's transcript. The orch never needs to continue its conversation on another machine: an orch started on a machine that has no local transcript for the worktree launches fresh with the carry-over. Keep only the same-machine fix: an orch start on the machine that holds the transcript resumes that session and never falls back to a fresh launch because an older live process holds it (A6, A7).

Rationale: Operator, 2026-11-15: the orch shall not need to continue anything on a new machine. Also settles F12: no orch transcript goes to the public weft repository.

### Added after Discussion (parent, 2026-11-15): Orch up during a hold (F14)

Decision: The orch is up while an adopt or adopt-fix hold stands, as D8 has it: every verb the act rule guards refuses except on the fix task and on a landing past its parent-side merge, and the orch handles the fix task's halts. The board's 'no orch action until adopt is done' means no orch action on held runs.

Rationale: Operator, 2026-11-15, answering F14: yes, the orch may be up during a hold with those refusals.

## Constraints

- `PATTERN-owner-machine`: the owner stays on roots and beams, written only by `boardengine`'s setter, read through `hubgeom.ParentOf`.
  The rest of a Finalize past its parent-side merge finishes on the run's owner, the adopter after a takeover (D1, D4).
- `PATTERN-pair-remote`: the code branch still goes along with a pair's records push, now once the records side lands (D2),
  and a refused push still leaves a failed-push record and a notice.
  A catch-up's local pair removal (D6) goes through `internal/pairteardown`'s local mode and deletes no origin branch.
- `PATTERN-dobby-runs`: runs are still started only by the parent's dobby, which starts only with its orch bar `--no-orch`.
- `PATTERN-lym-files`: every new tracked file sits under `_lym` and every cache under `.lym`, both named in `internal/lymdirs`.
- `PATTERN-tolerant-records`: the new tracked records and the Bolt `machines` and `holds` keys read tolerantly and are written through `internal/state` or the board store.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the synchronous push, the takeover entry's push, the archive and the transcript ref run in `internal/fabricengine` through `gitexec` and `gitrepo`.
- `PATTERN-shuttle-provider-seam`: Claude's transcript paths and the mirror's content handling live in `internal/shuttleengine/claudeengine`.
- `PATTERN-child-processes`: the mirror is a logged, registered detached child; the orch's Claude process registers in `internal/runproc` (D9), and D9's ends go through it.
- `PATTERN-daemon-wakeups`: D2's and D6's retry waits wake at most once a second and back off while idle.
- `PATTERN-refusal-way-forward`: every refusal the act rule, the holds, D3's owner refusal, D5's clash refusal and D9 add
  names its way forward and a test reaches it;
  none an agent reads names `lym adopt --release`, which agents' settings deny (D8).
- `PATTERN-shell-guard-table`: the deny of `lym adopt --release` is a row of `internal/shellguard`'s table.
- `PATTERN-code-conventions`: the mirror interval, the orphan grace, the code-tip grace and the chunk size are each one named constant.

## Auto-mode assumptions

- The mirror interval is 10 minutes (D10), the code-tip grace 30 minutes (D4) and the orphan grace 24 hours (D11).

## Open risks

- D10 pushes the orch's conversation to the weft repository, which is public (D10's Facts).
  The parent decides whether the operator allows the orch's transcripts there.
- D2 stops every run while origin is unreachable. The parent confirms the operator accepts no offline progress.
- D8's `lym adopt --release` lets the hub go on with failing tests, against the operator's "until every failing test it found is fixed".
  The parent confirms the operator wants that escape for a check that cannot pass on this machine.
- D8 brings the orch up while a hold stands, with every run verb refused bar the fix task's and a landing past its parent-side merge; the entry lists "no orch action" beside "no run starts".
  The parent confirms that reading.

## Acceptance criteria

- D1: for each state, a fixture of tracked records alone reads that state through `internal/runfsm` (new), with every `.lym` cache absent; a fixture meeting two states' conditions reads the one earlier in the order.
- D1: a slug with no batten record and a board status `done` reads `landed`.
- D1: the act rule answers `hand_over` for a run another machine owns, a landing past its parent-side merge included; `act` for one `landing` past its parent-side merge under a hold unless a push is owed offline or a pause covers it; `wait` under a hold, a pause, the state `paused` or `cancelling`, an owed push with origin unreachable, or a last transition whose code tip origin's code branch lacks; and `act` otherwise.
- D1: the dobby's pass, a hand `lym batten run`, `lym loom resume` and `lym tender resume` each refuse a run the act rule answers `wait` or `hand_over` for, naming the way forward.
- D2: a shed transition in a task pair pushes its records side, then its code side once the records side landed, and starts the next producer only after both landed on origin.
- D2: a pause written between a transition's commit and its push leaves that transition on origin, and the run's next producer does not start.
- D2: with origin unreachable, a transition leaves the run waiting with the push owed, starts no producer, and goes on once origin answers.
- D2: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns.
- D3: after `lym batten cancel` returns, the cancel intent is on origin in the run's batten record; a cancel interrupted on one machine reads `cancelling` on another.
- D3: `lym tender pause <slug>` puts the pause intent on origin and holds the run's processes here; a pause of a run another machine owns is refused naming that machine and, as way forward, the same verb there or `lym adopt` here first.
- D3: with origin unreachable, `lym tender pause <slug>` holds the run's processes here at once, and the pause intent is on origin once origin answers.
- D3: a finished cancel leaves the batten record with its finish on origin, and the run reads `cancelled` with its keep or remove.
- D3: a finished cancel's batten record has the shed state `cancelled`; the dobby's done-run clear leaves it and its task branch, and `lym batten cleanup` removes it once its pair is gone and refuses it, naming `lym fabric remove <slug>`, while its pair stands.
- D3: a pull-request record and the fixer attempt counts written on one machine are read on another after a pull.
- D3: a run whose `.lym` is removed reads the same FSM state as before the removal.
- D4: a records push rejected over a takeover entry naming another machine ends this machine's processes for the slug, pushes an `archive-owner/` tag holding its local commits and uncommitted files, and pushes nothing to the branch.
- D4: a push rejected over another run's commits on a shared records branch replays and pushes, and the run goes on.
- D4: a records push rejected over a takeover entry runs no replay, so no replay conflict stops the hand-over.
- D4: an adopter's takeover push rejected over the old owner's later transition writes the takeover entry onto origin's tip and lands, and the adopter reads the run at that transition.
- D4: after a takeover, a landing past its parent-side merge is driven on exactly one machine, the adopter, which finishes its Finalize.
- D4: after a takeover, the old owner's next transition leaves origin's code branch unchanged, and its code commits are under its archive tag.
- D4: a run whose last transition names a code tip origin's code branch lacks waits until that tip lands, then goes on from it; one still missing after the code-tip grace returns to its previous transition and runs the step again, and the owing machine's later push of that tip leaves origin's code branch unchanged.
- D4: an old owner past its parent-side merge that reads the takeover pushes nothing more to the parent's branch, and the adopter finishes the Finalize.
- D4: a release and a second adopt landing between an adopter's step 2 and step 3 stop the first adopter at its takeover retry, naming the release, and the second adopter drives the runs.
- D5: a registration whose former name is another machine's current name is refused naming that machine and `lym machine name --forget-former <name>`; after that verb, the registration lands.
- D5: an owner recorded as a former name of a registered machine reads as owned here on that machine and as that machine on another.
- D5: a machine's first registration carries the former names of its `machine.json`; `lym machine name <new>` puts the old name among them on origin when it returns.
- D6: a dobby that observes a sleep runs the catch-up before its next pass; a run another machine took meanwhile gets no pass, its processes end, and its local pair is removed with its origin branches left standing.
- D6: a run owned here whose local records are ahead of origin with its own commits has them pushed by the catch-up, and none is reset.
- D6: a run owned elsewhere at `Finalize` past its parent-side merge is handed over like any other run: its processes end here, and its owner finishes the Finalize.
- D6: with origin unreachable, the catch-up acts on no run and retries.
- D7: under an `adopt` hold no queued task is claimed and no run but a landing past its parent-side merge gets a pass, on the adopting machine and on another machine after its Bolt pull; under `adopt-fix` the fix task gets passes too.
- D7: a machine reading a hold holds its runs' processes through `hold` slug entries in its pause record, writes no hub entry, and drops only those entries once the hold clears, leaving an operator's hub pause standing.
- D7: under an `adopt-fix` hold the fix task's agent turns and network calls pass the gate, and the orch is not interrupted.
- D7: under a hold a run `landing` past its parent-side merge finishes, and the try count of an `adopt-fix` hold reads the same on every machine.
- D7: a standing `adopt` pause entry from before the upgrade becomes a pause intent on its run, which stays paused; a standing `owner` entry is dropped, and no code path writes either caller after the upgrade.
- D7: a standing `adopt` pause entry from before the upgrade writes one Bolt `adopt` hold naming this machine and posts one notice naming the hold, its machine and a re-run of `lym adopt` there, and no notice names `--release`; a landed run's standing entry is dropped, and a second machine's upgrade under a standing hold writes no hold.
- D7: `lym loom resume` on a worktree whose dobby is down refuses a run under a hold another machine wrote since that worktree's last Bolt pull.
- D8: `lym adopt <beam>` from a second machine's hub, with origin as the only shared state, ends with the beam's runs in the same FSM states as on the first machine and this machine as owner, with no `--keep-local` or `--drop-local`.
- D8: an adopt run on a machine with no local pair of a moved run adds the pair from origin before it writes the takeover entry.
- D8: two machines adopting at once end with one owner; the other's adopt refuses naming the winner.
- D8: a failing adopt check files one fix task, leaves an `adopt-fix` hold naming it, and exits non-zero; the task's green re-check clears the hold and the runs go on with no hand lift.
- D8: `lym adopt --release` clears a hold with the fix task standing; an agent's shell is denied it.
- D8: a hold written by a machine that then stops is cleared by `lym adopt --release` on another machine, whose adopt then goes on.
- D8: an adopt killed after its owner write goes on from the tracked state on its re-run and ends as an uninterrupted adopt does.
- D8: a transition the old owner pushed between the adopter's fetch and its takeover is on the adopter's local branches before the run acts, and the run goes on from it.
- D8: an adopt stopped by a release after its step 3 leaves a dobby up for each subject it owns; one stopped earlier starts none and names a re-run of `lym adopt`.
- D8: a registration D5 refuses at step 1 refuses the adopt with D5's way forward, and the adopt goes on after it.
- D9: an orch start whose session is held by this worktree's own earlier orch process, its strand already removed, ends that process and resumes the same session id.
- D9: an orch start whose session is held by a process lym has not registered refuses, naming the pid, and launches nothing fresh.
- D9: a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby running on this machine.
- D10: the transcript mirror of a fake transcript larger than the chunk size round-trips byte for byte through a fake origin, and a second mirror after an append uploads only the last chunk.
- D10: an orch start on a second machine with no local transcript restores origin's mirror into the project directory of its own worktree path and launches with `--resume` and the same session id.
- D10: an orch start whose session record names another machine restores that machine's mirror, also over a longer local transcript, and keeps the local copy beside it.
- D10: a mirror from a machine the act rule answers `hand_over` for writes nothing; the new owner's mirror whose lease is lost to the old owner's late mirror re-reads the ref and replaces it, and the old owner's mirror whose lease is lost to the new owner's writes nothing.
- D10: an old owner's late mirror never replaces the new owner's local transcript at its next start, and an old owner's orch start after the takeover writes no session record.
- D10: an orch start whose session record names another machine, with no mirror on origin, resumes its local transcript.
- D10: a turn end inside the mirror interval reaches origin once the interval ends.
- D11: a cancelled run's pair without keep is torn down by the dobby; a kept pair stays and is listed with its age in `lym beam status`, also after its board entry is removed.
- D11: an orphan with no commit and no live process is torn down no earlier than the orphan grace after its notice, its branches archived; one with a commit inside the grace stays.
- D11: `lym tender resume` pushes no pair.

## PATTERN changes

- `PATTERN-run-fsm` — adds: Reading or acting on a run's lifecycle state: only `internal/runfsm` (new)'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until an adopt's check is green. — added after Discussion (arbiter, 2026-11-15)
- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, both before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.

## Notes for the plan writer

- A split that raddle can follow, in dependency order:
  `internal/runfsm` (new) and its readers (D1); tracked records and caches (D3); synchronous push and the takeover entry (D2, D4);
  the registry (D5); the holds and the retired pause callers (D7); the catch-up (D6); adopt (D8); orch end, resume and transcript (D9, D10); leftover pairs (D11).
- The two-machine acceptance criteria need two hubs over one fake origin with different machine names; `hubforge` builds hub fixtures.
- `internal/adoptcli`'s package doc describes today's worker, record and verdicts at length; most of it retires with D8.
