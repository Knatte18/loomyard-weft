## Goal

One design of everything the beam `adopt-build` has left to build, on the code its two landed members left, for raddle to split again into board entries.

What is left: the synchronous push and the takeover, the machine registry, the catch-up, the holds, the foreground adopt and the leftover pairs,
plus the three members the operator added to the beam: the root beam in the board's output, the hub stop button and the restart launchers.

## Scope

In: the decisions below.
Each states its whole design, so the split can cut along any of them.

Out:

- What landed: `internal/runfsm`'s states, act rule, `runfsm.Judge` and `runfsm.Aside`; the tracked cancel and pause intents in the batten record;
  the orch's own-session resume, `runproc.RegisterOrch` and the subject-process end.
  A decision below fills or calls them and never redesigns them,
  bar D1's rename of one input and its owed push's last error, D5's hold machine, and the wording of their refusals.
- The orch transcript mirror and every cross-machine carry of an orch session (operator, 2026-11-15).
- A hub-wide pause shared between machines: `lym tender pause` with no slug, hibernate's pause and the hub stop stay machine-local.
- The split itself: raddle proposes it from this record.
- Claude account choice per worktree (`claude-account-pick-simple`) and the re-parent verb (note `move-task-by-handback`).

## Decisions

### D1 — A transition pushes its records before the run acts

- A shed transition commits its records as today, then pushes them synchronously: both sides in a task pair, records only in the prime.
  The records commit names the code tip the transition was made with (D2).
  The next producer starts only once every side has landed.
- The tracked writes `run-fsm-tracked-records` made push through the same synchronous path:
  the cancel intent and its finish, the run pause intent and its clearing, the pull-request record and the fixer attempt counts.
  Today each commits and spawns a detached push.
  `lym tender pause <slug>`, `lym tender resume <slug>` and `lym batten cancel` return with their intent on origin, or with the push owed when origin is unreachable.
- `fabricengine.PushPairAnchored` sends the records side first, and the code side only once the records side has landed;
  none when the records side fails or is rejected, so the order holds for every caller, `lym loom commit-records` included.
  Today it sends the code side first.
  A code side withheld this way is reported with the records side's failure, and the caller's next pair push sends it.
- A Finalize's parent-side merge pushes the parent branch before its transition's records and before the landing's board `done` (D2).
- Every detached status push that carries a run transition retires:
  the per-transition spawn behind `internal/statuscommit/statuscommit.go`,
  and the respawn of a pair push that exited on a held teardown lock (`internal/fabricengine/teardownlock.go`).
  A transition's push that meets a held teardown lock waits for its release.
  A wait past `fabricengine.StatusPushLockWait` turns the push owed, which releases the admission below.
  The detached pushes that stay lose the respawn too; the pair's next synchronous push carries their branch.
- The pending-flag respawn of fabric's records push lock retires too (`respawnForeignFlags`, run by `coalesceFlagged` in `internal/fabricengine/pendingflags.go`),
  so no side is pushed alone after a holder's release.
  A flag the holder does not owe stays on disk for its worktree's next push:
  in a pair, the owed retry of the push that set it, which sends both sides through `fabricengine.PushPairAnchored`, records first, with D2's takeover read.
- A transition takes a network gate admission (`pauserecord.Admit`) before its commit and releases it once every side has landed or its push turns owed,
  so a pause or hold written after the commit stops the next producer and never that push's first attempt.
  Fabric's pause deferral no longer applies to a transition's push; it stays for the detached pushes below.
- The anchor change's push (`internal/fabricengine/anchorchange.go`) and tender's prime push (`internal/tendercli/deps.go`) stay detached:
  neither carries a run transition.
- Every pair push that stays detached keeps `fabricengine.PushPairAnchored`'s order:
  the anchor change's `SpawnStatusPush`, fabric's commit spawn, `lym fabric sync` and `add`, and the deferred pair push a lift pays.
  The order moves into `coalescePushSides` (`internal/fabricengine/coalesce.go`), the step their detached child runs, which today pushes the code side first:
  it sends the records side first and the code side only once the records side has landed,
  and a rejected records side reads origin for a takeover (D2) before any replay and pushes no code side.
- A push that fails for a transient reason, such as an unreachable origin, leaves the run waiting at that transition with the push owed.
  It retries on a backoff and waits at the gate like any network call, so a pause holds it.
  `pairteardown.ReadRun` reads the owed push, offline or on a held teardown lock, into `runfsm.Inputs`' `PushOwed`, renamed from `PushOwedOffline`, which nothing fills today;
  its place in the act rule stays.
  Its refusal names the owed push and its last error, which `runfsm` carries in its inputs and verdict, and its way forward is a re-run once the owed push lands,
  in place of today's text that origin is unreachable and its re-run once origin answers.
  `lym beam status` shows the owed push and its last error; the failed-push record and its creel notice stay.
- With origin unreachable, `lym tender pause <slug>`, `lym tender resume <slug>` and `lym batten cancel` act here at once and owe their push;
  `runfsm.Aside` already sets an owed push aside for them.
  `lym batten cancel`'s output then names a re-run of `lym batten cancel <slug>` once origin answers, which pushes the intent and finishes the cancel.
- A rejected push reads origin's tip for a takeover first (D2), and replays the records as today only when it finds none.
- A Bolt write that carries a decision pushes synchronously before its writer goes on: a run claim, a landing's board `done`, an owner, a hold (D5) and the registry (D3).
  A board write that only displays a run status stays on the detached board sync.

Bound: no run acts while origin is unreachable, and an owed push holds the run at its transition until it lands.
Work an agent made inside a step before its transition is on this machine only; a stop there loses nothing pushed, and the step runs again from its start.
Until an owed intent push lands, another machine reads the run unpaused or not cancelled.

Rationale: the operator requires the whole state on GitHub at all times; today a push can lag or fail with nothing waiting for it.
A push costs two network round trips or more (`internal/fabricengine/pushclassified.go`), small beside an agent step.

### D2 — A takeover entry in the run's own records hands the run over

- Adopt (D6) writes a takeover entry, naming the new owner machine, into each moved run's batten record and child record, each on its own branch, and pushes it.
- When a run's records push is rejected, for any caller of the pair push, lym fetches the branch and reads the run's own record from origin's fetched tip before any replay.
  The caller of the pair push hands `internal/fabricengine`, which reads no board, the names that read as this machine:
  its current and former names from `machinename.Names` and the registry (D3).
  The takeover read compares the entry's machine against that set.
  When it holds a takeover entry naming a machine outside that set that no hand-over mark of this machine names, the run hands over without replaying:
  - it pushes no code side for the rejected push;
  - it ends its processes for the slug through `pairteardown.EndSubjectProcesses`, with the dobby end unset when the caller is that dobby's own run;
  - it pushes the code tip the run's last transition on origin names when origin's code branch lacks it, since that transition stands;
  - it archives the local tips as `fabricengine.ArchiveOwnerTips` does, uncommitted files first committed onto the archived tip,
    and its local parent branch's tip when this run's parent-side merge alone puts that branch ahead of its upstream, then resets that branch to its upstream;
  - it pushes the archive tags and posts one notice naming the machine and any pause or cancel intent it dropped;
  - it removes the local pair through `internal/pairteardown`'s local mode once its tips are archived, as D4 step 4 does;
    when the caller runs inside the pair, it leaves the removal to the parent's dobby (D7);
  - it writes this machine's hand-over mark, naming the slug and the takeover entry, once its archive tags land and before the local removal:
    a machine-local file under the hub's own `.lym`, keyed by slug and named in `internal/lymdirs`, so a pair removal does not take it.
    An adopt's takeover entry naming this machine (D6 step 3) drops this machine's hand-over marks of that slug.
  It pushes nothing more for the run.
  A later rejection reading a takeover entry a hand-over mark names skips the hand-over and pushes nothing for the run.
- A run `landing` past its parent-side merge hands over too, and its new owner resumes the Finalize from origin's records.
  The Finalize reads the owner from a fetched Bolt before its parent-side merge and before each later push, and hands over when another machine owns the run.
  After the merge into the local parent branch, the Finalize pushes the parent branch, then commits and pushes its transition, then writes the board `done`;
  today it marks the board `done` before the parent push (`internal/landingshed/finalize.go`).
  Under the push skip the board `done` follows the local merge.
  A takeover between the local merge and the parent push leaves origin's parent branch without the work: the adopter merges again, and the old owner archives and resets its local parent branch as above.
  A parent push rejected over an origin parent branch that already holds the task's work archives the local merge, resets the local parent branch to origin's, and goes on past the merge.
  A Finalize cut on its own machine between the local merge and the parent push, by a crash or a stop (D9), pushes the local parent branch at its next pass before its transition, in the same order.
- Until origin's code branch holds the tip the run's last transition names, `pairteardown.ReadRun` sets `runfsm.Inputs`' `CodeTipMissing` and the act rule answers `wait`;
  `lym beam status` names the tip and the machine that owes it.
  That machine's next hand-over or catch-up (D4) pushes it, and the owner's dobby fast-forwards its local code branch to it before the run's next pass.
  A tip still missing after the code-tip grace is given up: the owner commits and pushes a records commit that returns the run to its previous transition, and the step runs again.
  A machine owing a tip pushes it only while the run's last transition on origin names it, and with a lease on origin's code branch at the value it read with the records.
  After the push it reads the records tip again;
  when the run's last transition no longer names the tip, it resets origin's code branch with a lease to the value it read, so a given-up tip does not stay on origin.
  The owner guards it too: its code-side push rejected over a tip that no transition of the run on origin names resets origin's code branch with a lease at that tip, then pushes,
  so a tip left by an owing machine that died before its re-read goes too.
- A rejection whose fetched record holds no takeover entry, such as one over other runs' commits on a shared records branch like `main-weft`, replays and pushes as today.
- An adopter's own takeover push that is rejected fetches, then re-reads its hold and the subjects' owner from a fetched Bolt.
  When its hold is gone or either names another machine, it stops, naming the subjects' owner.
  Otherwise it writes the takeover entry again onto origin's tip and pushes again; it never replays its own commit and never hands over.

Bound: transitions the old owner pushed before the takeover landed stand, and the adopter goes on from them.
The old owner's step in flight runs to its next transition, which the takeover rejects; its result is archived under a tag.
An owed push that reaches origin after the takeover is rejected and archived by the hand-over, and its step runs again on the adopter.
A pause or cancel intent owed on the old owner when the takeover lands is dropped, and the notice names it to repeat on the owner machine.

Rationale: a takeover entry in the run's own record is a compare-and-swap per run that holds on records branches many runs share, and costs no extra fetch per transition.

### D3 — A machine registry in Bolt gives each machine one identity

- `board.json` gains a top-level `machines` map, keyed by machine id: each registered machine's current name and the former names it replaced.
  A machine id is a random id lym writes into `machine.json` beside the name at the machine's first registration; it tells two machines apart whatever they are named.
  Every writer of `machine.json` keeps the id: `machinename.File` carries it, and `lym machine name`'s rewrite copies it.
  Only `boardengine`'s registry setter writes the map, at a registration or an owner write, pushing synchronously.
- A machine registers at its first catch-up (D4) or dobby start after the upgrade, at its first owner write and at `lym machine name`, copying the former names of its `machine.json`.
  A registration refused at a catch-up or dobby start stops it: the machine acts on no run, posts one notice naming the refusal and its way forward, and `lym beam status` shows it.
  A machine whose `machinename.FilePath` resolves no directory has no file to hold an id, and its registration is refused,
  naming the variable to set: `XDG_CONFIG_HOME` or `HOME`, or `LOCALAPPDATA` on Windows.
  A registration whose id the registry holds under a current name that is neither this machine's current name nor one of its former or forgotten names is refused as a copied `machine.json`;
  its way forward is `lym machine name <new>` on this machine.
  `lym machine name` runs the same check on each hub it sweeps before it writes the file:
  when it finds this machine's id under such a name, it writes a fresh id with the new name; otherwise it keeps the id.
  A registration whose former name is another registered machine's current name is refused, naming that machine;
  its way forward is `lym machine name --forget-former <name>` (new) on this machine, which drops that former name from its `machine.json` and its registry entry.
  The dropped name moves to the file's forgotten names, which only the copied-file check reads.
  `--forget-former` lists every owner on the fetched board, and every takeover entry and unfinished cancel intent of a run whose pair stands here, that names the dropped name,
  with `lym adopt` of each subject as the way to take it back once the registration lands.
  A registration whose current name is a former name of another registered machine is refused, naming that machine;
  its way forward is `lym machine name <new>` on this machine, which records a name a registration refused among none of this machine's former names.
  A registration whose current name is another registered machine's current name is refused the same way.
- `lym machine name` records the rename in the registry under the same refusals, besides its sweep of owners (`internal/machinecli/set.go`).
- Every comparison of a recorded machine name with this machine goes through the registry: a former name of a registered machine reads as that machine.
  A comparison resolves the recorded name to a registry entry and reads it as this machine only when that entry's id is this machine's id;
  a machine with no registry entry reads no registered machine's current or former name as its own.
  `boardengine.OwnedElsewhere` takes the registry, every owner write writes the current name,
  and the comparisons that name the machine directly move too: `dobbyengine.OwnedWorktrees` and `dobbyengine.EnsureLive`.
- `internal/machinename` stays this machine's only name source.

Bound: `--forget-former` changes only this machine's own file and entry.
From the other machine's registration on, every record under the clashing name reads as that machine on every machine, whether this machine forgets the name or not:
an owner this machine's rename sweep missed hands its subject to that machine with no adopt and no takeover entry, and this machine's work on it reaches the owner only through D2's archive tags.
A registration clash refuses and never overwrites another machine's entry.
A machine whose `machine.json` was lost gets a new id, so its old entry refuses its old name: it registers under a new name and adopts its own subjects again, and its old entry stays.
A hold its old entry wrote reads as another machine's, so its own adopt is refused, and the operator moves the hold with `lym adopt --take-hold`.
Two machines sharing one copied `machine.json` read as one machine until either renames; the other's next registration is then refused as a copy.
A machine whose user config directory resolves from no variable runs no dobby and no catch-up until the variable is set, where today it runs under the hostname rule.

Rationale: `osl46` and `osl-1046` are one machine before and after `lym machine name`, and only that machine's `machine.json` knows it.

### D4 — After an absence, lym catches up from origin before it acts

An absence is a sleep a long-lived lym process observes through `pauserecord.SleepWatch` over `pauserecord.SleepTolerance` (the dobby loop or the orch watcher),
a worktree's dobby starting, or a hand pass, whose catch-up runs over its own run only.

Before any act rule answer after an absence, the catch-up runs:

1. It fetches Bolt, replays local display commits as the board sync does, and reads the owners, the registry and the holds, registering this machine when the registry has no entry of its id (D3).
2. It fetches each local pair's branches and reads each run's record from the result.
3. For a run this machine owns, local commits on top of origin's tip are owed pushes, and it pushes them; a takeover entry naming another machine is D2's hand-over.
4. For a run another machine owns, a `landing` past its parent-side merge included, it first pushes the run's owed commits, records first, when origin's record holds no takeover entry.
   It then ends this machine's processes for the slug through `pairteardown.EndSubjectProcesses`,
   pushes the code tip the run's last transition on origin names when origin lacks it, archives the local tips as D2 does and pushes the archive tags,
   posts D2's one notice naming the machine and any pause or cancel intent it dropped, writes D2's hand-over mark once its archive lands,
   and removes the local pair through `internal/pairteardown`'s local mode, which takes no origin step and writes no board.
5. It writes this machine's pause record entries from the pause intents of the runs it owns, and drops the caches of the runs it touched; each run's next pass is a full pass.

- The dobby's periodic Bolt pull (`internal/dobbyengine/handover.go`) stays, and a hold it reads applies at once (D5).
- The dobby's own handover of a subject calls `pairteardown.EndSubjectProcesses` with the dobby end set; a catch-up the dobby runs leaves it unset.
- The origin-owed marker (`internal/battencli/resumereconcile.go`, read by the dobby's passes and batten's continue-pass check) retires: the catch-up is the reconcile it stood for.

Bound: a catch-up that cannot reach origin acts on nothing and retries on D1's backoff; `lym beam status` shows it waiting on origin.

Rationale: the operator requires lym to check pushed state before acting after the machine slept, hibernated or was off;
the dobby detects a sleep today (`internal/dobbyengine/loop.go`) but only owes full passes and pulls Bolt up to 5 minutes later.

### D5 — Adopt holds live in Bolt; the owner and adopt pause callers retire

- `board.json` gains a top-level `holds` list.
  A hold has a kind, `adopt` or `adopt-fix`, the machine, the time, its adopt's subjects and each subject's owner before the adopt,
  and for `adopt-fix` the fix task's slug and each subject's try count.
  Only `boardengine`'s setter writes it, pushing synchronously.
- A hold covers the whole hub: the act rule answers `wait` for every run on every machine.
  `pairteardown.ReadRun` sets `runfsm.Inputs`' `Held`, false for the `adopt-fix` hold's fix task; the act rule already lets a `landing` past its parent-side merge act under a hold.
  The refusal of a held run names the hold's machine, so `runfsm` carries it in its inputs and verdict;
  its way forward is waiting for the hold to clear, or asking the operator when that machine has stopped.
- Every verb that asks the act rule outside the dobby's pass, such as `lym loom resume` and `lym tender resume <slug>`, fetches Bolt first when origin answers.
- A machine that reads a hold writes no hub entry.
  It writes a slug entry under one new `pauserecord` caller `bolt-hold` for each of its runs the act rule answers `wait` for under the hold,
  none for the `adopt-fix` hold's fix task, and drops those entries once it reads no hold.
  The caller's constant is `pauserecord.BoltHoldCaller` (new), apart from the legacy `pauserecord.HoldCaller` (`adopt-hold`) the upgrade still reads.
- `pauserecord`'s own readers read a `bolt-hold` slug entry as a hold, and the entry names the hold's machine.
  `pauserecord.CoveredError` words it as held by an adopt on that machine, with the way forward above, and never names `lym tender resume <slug>`.
- The hold does not hold an owed push (D1): a run whose push is owed gets no `bolt-hold` entry,
  and a machine drops a run's `bolt-hold` entry once that run's push turns owed, so its retries pass the gate.
  The act rule still answers `wait` for it, so no producer starts.
  Nothing orders that push before an adopter's takeover; D2's bound covers one that lands after it.
- The `bolt-hold` entries hold only those runs: the orch, `lym adopt`'s own process and its deploy and check children, and the operator's `lym batten cancel` go on.
  The orch is up while a hold stands (operator, 2026-11-15).
- The `bolt-hold` entries keep apart from the operator's hub entry and hibernate's: dropping them drops only the `bolt-hold` caller's,
  and the hibernate wake never reads a `bolt-hold` slug entry as a hub entry.
  A slug with a live entry gets no `bolt-hold` entry.
  `lym tender resume` lifts no `bolt-hold` entry while the Bolt hold stands; lifting a slug's own entry under a standing hold writes the `bolt-hold` entry in its place.
  `lym tender resume <slug>` of a slug only a `bolt-hold` entry covers refuses, naming the hold's machine and the way forward above.
- A run another machine owns is not paused: the act rule answers `hand_over`, and `pairteardown.EndSubjectProcesses` ends its processes.
  The `owner` caller retires with every writer: the dobby's owner pause (`internal/dobbyengine/owned.go`), `tendercli.PauseOwnedElsewhere` and the process end beside it,
  and the hibernate wake's hold of runs owned elsewhere.
  The `adopt` and legacy `adopt-hold` callers' writers retire with adopt's pause step (D6); the two names stay declared only for the upgrade's first read.
- At the first read after the upgrade, a standing `adopt` or `adopt-hold` entry of a run this machine owns in a state that is not terminal becomes a run pause intent, so no run resumes unasked.
  Every other entry of those callers, and every `owner` entry, is dropped.
  The first read pushes the pause intents and the Bolt hold first, and only then drops the `adopt` and `adopt-hold` entries, so a cut first read runs again from the entries left.
  A first read cut between its hold push and its drops, whose hold a green check cleared meanwhile, writes the hold again at its re-run, and a re-run of `lym adopt` clears it.
  When at least one entry became a pause intent or belonged to a released `queued` task, the first read writes one Bolt hold, `adopt` or `adopt-fix` (below), naming this machine and the subjects of those runs and tasks,
  and posts one notice naming the hold, its machine and, as way forward, a re-run of `lym adopt` of those subjects there,
  then, once that adopt's check is green, `lym tender resume <slug>` there for each run whose entry became a pause intent.
  When the first read finds an open adopt fix task of one of those subjects, the hold is `adopt-fix` naming it,
  each subject's try count read from `internal/adoptcli`'s local tries file (`fix-tries.json`).
  A first read that finds another machine's hold writes none, and its notice names that hold's machine.
- The dobby's handover and the orch watcher, which ask the adopt record whether an adopt here covers a worktree, read a hold this machine wrote instead, once D6 retires the record.

Bound: a machine other than the adopter learns of a hold at its next Bolt pull or catch-up; until then its dobby goes on for at most the dobby's pull interval, and D2's bound covers the transitions it pushes.
A run holding its parent lock gets no slug entry, so the gate does not hold its step in flight; the act rule still starts no producer for it.
A Finalize cut between its local merge and its parent push (D2) waits out a hold with its merge local, since `PastParentMerge` reads origin's parent branch;
its parent push follows once the hold clears, so no merge the adopt's check did not see lands under the hold.
A hold stands until a green check clears it, the adopt's or, under `adopt-fix`, the fix task's re-check;
a hold whose adopter stopped stands until the operator takes it over with `lym adopt --take-hold` (D6).
No refusal or notice an agent reads names `--take-hold`.

Rationale: the operator decided nothing happens in the hub until adopt is done and its failing tests are fixed, and the pause record is machine-local, so no other machine sees a hold kept there.

### D6 — `lym adopt` runs in the foreground: hold, owner, pull, takeover, check

`lym adopt <subject>...` runs every step in the caller's process and prints each step as it ends.
It starts no detached worker and writes no adopt record; each step reads tracked state, so a re-run goes on from where the state stands.

1. One Bolt commit writes an `adopt` hold, this machine as the subjects' owner and this machine's registry entry (D3), and pushes it.
   A hold another machine wrote refuses the adopt, unless `--take-hold`, naming the writer machine and the hold's time, with D5's way forward.
   A rejected push reads Bolt again: an owner of these subjects or any hold another machine wrote since refuses naming that machine; otherwise it replays and pushes.
   A registration D3 refuses refuses the adopt with D3's way forward.
   A hold this machine wrote for the same subjects, the same set, stays as it stands, its kind, fix task and try counts included:
   the commit writes only the owner and the registry entry, and the adopt goes on from step 2.
   One this machine wrote for any other set of subjects, an overlapping one included, refuses the adopt, naming the hold's subjects;
   its way forward is `lym adopt <hold's subjects>` first.
   Under its own `adopt-fix` hold, as under `--take-hold`, a failing check files no second task while the fix task stands.
2. For each moved run it fetches the run's branches, archives local tips origin lacks as D2 does, resets them to origin's, and adds the pair from origin when this machine has none.
   A run is no moved run when the hold names this machine as its subject's owner before the adopt and origin's record of the run holds no takeover entry naming another machine:
   steps 2 and 3 skip it, and step 4's catch-up pushes its owed commits.
   The hold keeps that owner across a re-run, so an adopt killed after its owner write still moves the runs it took.
3. It writes the takeover entry on each moved run's records branches and pushes it (D2), then fast-forwards the local records branches and code branch to origin's tips.
4. The catch-up (D4) over the whole hub.
5. The deploy, as today.
6. The hub workspace file regenerated.
7. The check: the hub's `adopt` gate in each subject's worktree, in the foreground, its output on the terminal.
8. A green check clears the hold, drops this machine's `bolt-hold` pause entries at once, and runs `orchcli.BeamUp` for each subject; the runs go on by the act rule,
   bar a run a pause intent holds, such as one the upgrade paused (D5), which waits for `lym tender resume <slug>`.
   A failing check files a fix task through the fix loop (`internal/adoptcli/fixloop.go`, `fronttask.File`), replaces the hold with an `adopt-fix` hold naming it,
   runs `orchcli.BeamUp`, and exits non-zero naming the task.
   The task's landing runs the check again: green clears the hold, failing files the next task within `adopt_fix_tries`.
   Each subject's try count lives in the `adopt-fix` hold, so a re-check on any machine reads the same count.
   At the limit the hold stays, and the dobby's notice names the failing tests and the fix task.

`lym adopt --take-hold <subject>...` (new) takes over a hold another machine wrote.
It refuses a hold whose subjects are any other set, an overlapping one included, naming the hold's subjects.
Otherwise its Bolt commit rewrites the hold to name this machine, keeping kind, fix task, try counts and the owners before the adopt, writes this machine as owner, pushes, and runs steps 2 to 8.
Under `adopt-fix` the fix task stands and a failing check files no second task.
Every adopt re-reads its own hold from Bolt before each step from step 2 on, and stops once the hold is gone or names another machine, naming the subjects' owner and starting nothing.
A shell-guard row denies `--take-hold` to every agent; adopt's help names it.
The row's steer and notice say only that an adopt's takeover of another machine's hold is the operator's, the steer naming asking the operator as the way forward, and neither spells the flag;
its pattern and its `Forms` entry, which the no-denied-recovery scan reads and no agent is shown, are the only places the row names it.

Kept: `--archives`, `--no-orch`, `--all` and `--here`.
`lym adopt check` stays, run by the fix task's landing through `fronttask`'s `adopt-check` row:
it reads the `adopt-fix` hold's try counts, and on green clears the hold and runs `orchcli.BeamUp` for each subject, as step 8 does.
`--here` stays behind `lym s` and keeps its fast outcomes: `no_subject` in a worktree with no subject,
and `owned_here` after one Bolt pull when no other machine owns the subject, writing no hold and starting nothing.
Retired: `--keep-local`, `--drop-local`, `--discard`, `lym adopt worker`, the adopt record, the held-back project verdicts, the adopt pause step and every writer of the `adopt` and `adopt-hold` callers.
`internal/adoptcli`'s and `internal/adoptrecord`'s package docs are rewritten to the foreground flow, or the package removed with the record.
`lym beam status` replaces its adopt section, read today from the adopt record (`internal/beamcli/cli.go`), with the Bolt hold:
its kind, machine, time and subjects, the fix task and try counts under `adopt-fix`, and for this machine's own hold the `lym adopt <subjects>` re-run.

Bound: adopt works from pushed state alone; the old owner's unpushed work reaches the adopter only through D2's archive tags, which `--archives` lists.
`--take-hold` skips no check and lets no run go on before a green check: it moves the hold and its adopt to this machine.
The shell-guard row denies the plain command form to agents, but it is a guardrail over static text that a written script or a quoted command substitution passes (`internal/shellguard/doc.go`);
a bypass is bounded by the rest: no check skipped, no run before a green check, and the old adopter stopping at its next hold re-read, naming this machine.

Rationale: the operator decided adopt runs in the foreground and that nothing happens in the hub until it is done and every failing test it found is fixed;
D1 and D2 leave no local state for adopt to reconcile.

### D7 — Leftover pairs are torn down or reported

- The dobby tears down, through `internal/pairteardown`, the pair of each run in the FSM state `landed`, or `cancelled` without keep,
  with no pause or hold covering it and no other machine owning it; a cancelled run's batten record goes in the same step.
  This widens the ended-run teardown of `internal/dobbyengine/ended.go`, whose stranded-run condition reads a board entry that ended and so takes neither case today;
  the widened condition reads the FSM state.
- The dobby also removes, through `internal/pairteardown`'s local mode, the local pair of a run a hand-over mark of this machine names (D2) while another machine owns the run,
  under a hold too, since the local mode takes no origin step.
- A cancelled run whose pair was kept stays: `lym beam status` lists it with its age, and each dobby start posts one notice listing kept pairs and naming `lym fabric remove <slug>`.
- A run cancelled with keep before the upgrade keeps its batten record while its pair stands (`internal/battencli/cleanup.go`), in shed state `cancelled` with no cancel intent;
  `runfsm.Derive` reads `Cancelled` only from a cancel intent with a finish (`internal/runfsm/state.go`), so such a run reads neither cancelled nor kept.
  The first dobby start after the upgrade gives each such record whose pair stands here a finished cancel intent with keep, naming this machine, pushed through D1's synchronous path.
- A pair with neither a board entry nor a batten record is an orphan.
  The dobby posts a notice at first sight saying when it goes, and tears it down no earlier than the orphan grace after that notice,
  only when neither branch has had a commit for the orphan grace and no process of its slug lives; uncommitted files are committed onto the archived tip first.
  The dobby records each orphan's first-sight time in a machine-local file under the hub's `.lym`, named in `internal/lymdirs`,
  written with the notice and dropped with the teardown or once the pair stops being an orphan, so a dobby restart neither resets nor skips the grace.
- `lym tender resume` spawns no pair push for a transition, since D1 pushes every transition synchronously;
  it still pays a detached push a pause deferred, such as an anchor change's.
- The hub workspace file is regenerated after each teardown (`fabricengine.RegenerateHubWorkspace`).

Bound: an orphan in use stays while it commits or runs a process; an idle one goes after the grace, its branches kept under archive tags.
A kept pair's batten record keeps it out of the orphans, also when its board entry is removed.

Rationale: a bench pair, a kept pair and a trial pair stood for days in the operator's VS Code with nothing reporting them.

### D8 — The board prints the prime as the root beam `root`

- `lym board list` and `lym board get` print `parent_beam` as `root` for every entry whose nearest beam is the prime, never empty.
  `root` is `boardengine.RootSubject`, the slug the board reserves and `lym adopt root` and `lym beam stop root` take.
- `lym board list --text` shows the root project as a `P0` row among the projects, with the title the board files give it.
  The JSON `tasks` array gets no root row.
- The mapping is at the printing boundary only: where the brief task's parent beam is set in `internal/boardengine/store.go` and where `internal/boardcli/cli.go` writes the `get` envelope.
  `boardengine.ParentBeam` and `hubgeom.ParentOf` keep returning the empty slug for the prime, since every owner read and beam verb takes it as the prime.
- The readers of the printed value move in the same change: `contracts/orchrole/orch-role.md` (the entries an orch reads and releases, and the parent orch's address,
  `<shortname>:orch` for `parent_beam` `root`), `plugins/ly/skills/board/SKILL.md`, and the `boardcli` help and package doc.

The entry's "every reader moves" is met for the readers of the printed value; the dobby's and the board verbs' code reads keep the empty slug on purpose.

Rationale: an empty value reads as "no beam", while the board view and window titles show the prime as `P0`.

### D9 — `lym beam stop --all` takes the hub down on this machine, behind a stop button

- `lym beam stop --all` (new flag) ends everything of the hub on this machine and writes no tracked record.
  It re-starts itself detached from its caller, through `selfexec` with its spawn logged, so the click's own session ending last does not cut it.
- Its sequence:
  1. A hub pause entry with a grace, then a confirmation that every session is idle, no network call is in flight and no land lock is held.
     The sequence is hibernate's: its pause-and-confirm in `internal/hibernateengine/watch.go` becomes one exported function that hibernate's carrier and the stop both call.
     The function owns only an entry it added; a hub entry already standing, the operator's or hibernate's, covers the stop and stays.
     A confirmation not reached within hibernate's own wait does not stop the stop: it goes on, and its output names what was still busy.
  2. Each task pair's processes and reed session, through `lym reed down`'s path, which accepts a pair a hub pause covers.
  3. For the prime and each local beam, `orchcli.BeamDown`: the orch strand, the orch's registered Claude process, the dobby and, for the prime, the coverage-map upkeep.
     `orchcli.BeamDown` gains the end of the orch's registered Claude process through `runproc.EndSubject`, as its last step,
     so every caller ends it too: `lym beam stop` of one worktree, the cancel of a beam and pawl's beam-down row.
     A caller inside that orch's own session has finished every other step when it ends.
  4. The reed sessions of the beams.
  5. Run processes no run owns, as `tenderengine.Reap` ends them.
  6. Its own pause entry, if it added one, lifted by identity through `tenderengine.Lift`.
     The lift pays the detached pushes a pause deferred, as every lift does; those pushes run to their end after the stop.
  7. The prime's reed session, last.
- It ends every worktree of the hub on this machine, whoever owns it, and leaves a standing Bolt hold as it is.
- Reed's default `prime` button group, in `internal/reedengine/template_posix.yaml` and `internal/reedengine/template_windows.yaml`, gains a `stop` entry running `lym beam stop --all`,
  with a `confirm` text saying it stops every orch, dobby, run and reed session of the hub on this machine.
  The `prime` group shows only in the prime orch's session.
  A hub that sets `buttons.prime` in its own layer replaces the group whole and adds the entry by hand.
- After a stop, the restart (D10), `lym beam start` or `lym s` brings the hub back; a run whose loop was ended halts `loop-exited` at its first pass, and the orch settles it, as after a crash.

Bound: the stop ends agent steps mid-step; D1 leaves every transition pushed or owed, so nothing pushed is lost and the cut step runs again.
A transition whose push is owed stays local, and D4's catch-up at the next dobby start pushes it.
It writes no pause intent and no tracked record, so no other machine reads anything from it, and its own pause entry is gone when it ends.
A stop that dies midway leaves its pause entry; `lym tender resume` lifts it.
The operator closes terminal windows by hand.

Rationale: the operator stops a machine today by `lym beam stop` per worktree, `lym reed down` per pair and `lym tender reap`.
The entry builds the stop on the run intents so that it leaves every run at a transition pushed or owed;
D1 gives that state, and a per-run intent would be pushed and read as `paused` on other machines.
The stop lifts its own entry because hub-wide `lym tender resume` restarts nothing itself: only a live dobby's pass restarts a run, and the stop ended every dobby.

### D10 — `lym machine start` brings back every hub this machine owns, from desktop launchers

- `lym machine start [<shortname>]` (new) starts one hub, or every hub `hubregistry.Live` lists, from any directory.
- For each hub it takes the worktrees this machine owns from `dobbyengine.OwnedWorktrees` over the hub's prime, whose owner reads go through the registry (D3), filtered per worktree:
  - the prime or a beam another machine owns is skipped, naming that machine;
  - a prime with no recorded owner is skipped and says so, since `dobbyengine.EnsureLive` would claim it; the hub's owned beams still start;
  - the rest start through `lym beam start` in the hub's prime as a child process, the prime with `--all` when it is owned here, each beam by its slug otherwise.
  It never starts a hub through `lym s`, whose `lym adopt --here` takes over a hub another machine owns; the terminals run `lym reed attach` for the same reason, where the entry had `lym s`.
- It opens the hub's VS Code workspace file, and for each worktree it started a terminal outside VS Code running `lym reed attach`, through `termwindow.Resolve`.
  `internal/termwindow` gains native Windows (`wt.exe`).
  Where no terminal launcher resolves, the verb prints each worktree's attach command instead.
- It resumes no run: the orchs and dobbies take the runs up.
- When a hold stands on the hub it prints the hold's machine and subjects, and for this machine's own hold the re-run `lym adopt <subjects>`.
- Desktop launchers call the verb: one per hub and one for every hub.
  A new package `internal/desktoplauncher` (new) writes them: a `.desktop` file in the freedesktop desktop directory on Linux, a `.cmd` file on the Windows desktop, which under WSL runs the verb through `wsl.exe`.
  lym adds a hub's launcher, and the all-hubs one with the first such hub, when any dobby of the hub starts on this machine for a worktree this machine owns.
  It removes a hub's launcher when this machine's verb, dobby or catch-up (D4) finds that this machine owns nothing in the hub,
  and the verb removes each launcher on disk whose hub `hubregistry.Live` no longer lists; the all-hubs one goes with the last hub.
  A beam gets no launcher of its own.

Bound: the verb starts only what this machine owns, and never claims a prime; it writes no tracked record.
A launcher's removal waits until this machine next runs the verb or a dobby, since no other machine reaches this desktop.

Rationale: after a crash the operator finds the hubs and beams that were in flight by hand, and starting them is what the orchs and dobbies cannot do for themselves.

### Added after Discussion (parent, 2026-11-16): The hub stop writes no per-run pause or cancel intent

Decision: D9 stands as written: the hub stop (lym beam stop --all) writes no per-run pause or cancel intent, and its guarantee that every run is left in a pushed state rests on D1's synchronous pushes. Drop the Open risks line on this.

Rationale: Operator decision in chat, 2026-11-16. A pushed per-run intent would read as paused on every other machine, against the entry's statement that the stop needs no shared pause between machines.

### Added after Discussion (parent, 2026-11-16): Only the printed parent_beam names the prime root

Decision: D8 stands as written: the prime maps to root only where parent_beam is printed (lym board list and get, the --text P0 row) and in its prose readers (orch-role contract, board skill, boardcli help); the code reads through boardengine.ParentBeam, Board.ParentBeamOf and hubgeom.ParentOf keep the empty slug for the prime.

Rationale: Operator decision in chat, 2026-11-16. The operator's complaint was what the board shows; moving the internal value would change every owner reader for a display fix.

### Added after Discussion (arbiter, 2026-11-16): Round-2 review accepted at the hard cap: every finding answered in the record

Decision: The Discussion review's round-2 findings F1 to F17 are each answered by the round-2 fixer's edits now in the record (D1's coalescePushSides order and anchor-change criterion; D5's bolt-hold reading by pauserecord's readers and CoveredError wording; D3's id-matched comparisons, id lifetime, copied-file refusal and --forget-former reach; D2's name set handed to fabricengine and hub-rooted hand-over mark; D5's dropped ordering claim and D4 step 4's records-first push; D6 step 2's owner-before-adopt test; D5's upgrade order; D6's lym adopt check and same-set rule; D7's first-sight file; the Scope, D9 and criterion wording). The segment is accepted unconverged at the hard cap. The parent entry 'The hub stop writes no per-run pause or cancel intent' reads its 'every run is left in a pushed state' as D9's bound states it: every run at a transition pushed or owed, an owed one pushed by D4's catch-up at the next dobby start.

Rationale: The round is at the perch's hard cap; an accept is allowed where the fix in place answers each open finding, which a check of the record and the cited code (coalesce.go, pendingflags.go, pauserecord/record.go CoverageOf, whose Held only battencli/dobbyreads.go's retiring owner probe reads) confirms. The pushed-or-owed reading follows D1's bound and the fixer's deferred consistency note, and changes no decision.

### Added after Discussion (arbiter, 2026-11-16): Split review round 2 accepted at the hard cap: both findings answered in the proposal

Decision: The split review's round-2 findings are each answered by the round-2 fixer's edits now in _lym/raddle/proposal.yaml, and the Raddle-Bouncer segment is accepted unconverged at the hard cap. F1 (criteria-built-by-later-part): the three criteria moved whole out of transition-push-takeover: the D1 hold-write bullet to machine-registry-holds-catchup, the D2 rejected-takeover-push bullet to adopt-foreground, and the D2 in-loop hand-over local-pair bullet to leftover-pairs-teardown. Each of those parts has transition-push-takeover on its depends_on path. F2 (fix-tries-reader-route): the fix-tries.json reader and writer move from internal/adoptcli/holds.go into a new internal/fixtries as fixtries.Read and fixtries.Write, importing only internal/state. The upgrade's first read runs at the dobby start in internal/dobbyengine, before its catch-up, and calls fixtries.Read on the subject directories its caller hands in. dobbyengine's import allowlist gains internal/fixtries. adopt-foreground's takes and delivers name the same package and functions. The file D5 names (fix-tries.json under the adopt scratch directory) is unchanged; only the Go package that reads it moves.

Rationale: The round reached the perch's hard cap with both findings open in the ledger, but the fixer ran after the review (review 11:17, proposal and fixer report 11:20, judge 11:20:48), so the judge ruled on the review's findings, not on the edited proposal. Checked against the code: holds.go's readTries and writeTries import only fmt, path/filepath and internal/state, so internal/fixtries forms no cycle. Only internal/beamcli and cmd/lym import internal/adoptcli, and internal/adoptcli imports internal/dobbyengine, so the first read could not call into adoptcli; the new package removes that route. The dobby start as the first read's place matches the record's export line, which pairs the hold setter with the upgrade's first read and the catch-up with the dobby start. The record does not fix D5's reader package, so moving it changes no decision.

## Constraints

- `PATTERN-owner-machine`: owners stay on roots and beams, written only by `boardengine`'s setter, read through `hubgeom.ParentOf` and the registry (D3).
  The rest of a Finalize past its parent-side merge finishes on the run's owner, the adopter after a takeover (D2).
- `PATTERN-pair-remote`: the code branch goes along with a pair's records push, after the records side (D1); a catch-up's local pair removal and D7's teardowns go through `internal/pairteardown`.
- `PATTERN-dobby-runs`: runs are started only by the parent's dobby; the stop and the restart start dobbies only through `orchcli.BeamUp` paths (`lym beam start`).
- `PATTERN-lym-files`, `PATTERN-tolerant-records`: new tracked records under `_lym`, caches under `.lym`, both named in `internal/lymdirs`; the Bolt `machines` and `holds` keys and the takeover entry read tolerantly.
  Desktop launchers sit outside the hub and are named by `internal/desktoplauncher` (new) alone.
- `PATTERN-fabric-boundary`, `PATTERN-gitrepo-client-boundary`: the synchronous push, the takeover read and write, the archive and the catch-up's fetches run in `internal/fabricengine`.
- `PATTERN-child-processes`: every process end goes through `internal/runproc`; the stop's detached re-start and the restart's `lym beam start` child name their binary through `selfexec` and log the spawn.
- `PATTERN-daemon-wakeups`: the owed push's and the catch-up's retry waits wake at most once a second and back off while idle.
- `PATTERN-refusal-way-forward`: every refusal added names its way forward and a test reaches it; none an agent reads names `lym adopt --take-hold`.
- `PATTERN-shell-guard-table`: the deny of `lym adopt --take-hold` is a row of `internal/shellguard`'s table, whose steer and notice never spell the flag (D6).
- `PATTERN-cli-cobra`: `lym machine start`, `lym machine name --forget-former` and `lym beam stop --all` sit in their modules' cobra subtrees.
- `PATTERN-test-only-code`: every new function has a production caller in the part that adds it: the takeover reader in the rejected-push path, its test writer in `internal/testkit`,
  its production writer with adopt; the hold setter in the upgrade's first read; the catch-up at the dobby start.
- `PATTERN-code-conventions`: the code-tip grace and the orphan grace are each one named constant.

## Auto-mode assumptions

- The code-tip grace is 30 minutes (D2) and the orphan grace 24 hours (D7), as the Sketch set them.
- The prime prints as `root`, not `P0` (D8).
- The stop lifts its own pause entry before it ends (D9).
- The stop goes on past an unconfirmed idle (D9).
- The restart verb is `lym machine start`, and the hubs it starts are those `hubregistry.Live` lists, with no separate in-flight mark (D10).
- The stop's verb is `lym beam stop --all` (D9).

## Open risks

None.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-2-review.md; fixer report: _lym/reviews/discussion/round-2-fixer-report.md.
  - adopt-rerun-resets-adopt-fix-hold (class design, severity MEDIUM, open in rounds 1, 2)
  - bolt-hold-read-as-operator-pause (class design, severity BLOCKING, open in rounds 2)
  - forget-former-hands-records-to-another-machine (class design, severity MEDIUM, open in rounds 2)
  - hold-freezes-owed-pushes-unstated (class design, severity MEDIUM, open in rounds 1, 2)
  - machine-id-has-no-lifetime (class design, severity MEDIUM, open in rounds 2)
  - pending-flag-respawn-pushes-code-alone (class design, severity BLOCKING, open in rounds 1, 2)
  - registry-lets-two-machines-claim-one-name (class design, severity MEDIUM, open in rounds 1, 2)
  - takeover-read-cannot-see-former-names (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Discussion-Review -->

## Acceptance criteria

- D1: a shed transition in a task pair pushes its records side, then its code side once the records side landed, and starts the next producer only after both landed on origin.
- D1: a pause written between a transition's commit and its push leaves that transition on origin, and the run's next producer does not start.
- D1: a transition's push that waits on a held teardown lock past the push lock wait turns owed, and a pause written then holds the process's network calls.
- D1: with origin unreachable, a transition leaves the run waiting with the push owed, starts no producer, shows in `lym beam status`, and goes on once origin answers.
- D1: a push owed on a held teardown lock is refused naming the owed push and its last error, not an unreachable origin.
- D1: a code side flagged by a push whose lock wait gave up never reaches origin before its records side, nor after a takeover entry naming another machine.
- D1: an anchor change's push in a pair another machine took over leaves origin's code branch unchanged.
- D1: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns.
- D1: `lym tender pause <slug>`, `lym tender resume <slug>` and `lym batten cancel` return with their intent on origin, and a pull-request record write and a fixer attempt count are on origin before the next producer starts.
- D1: with origin unreachable, `lym tender pause <slug>` holds the run's processes here at once, and the pause intent is on origin once origin answers.
- D2: a records push rejected over a takeover entry naming another machine runs no replay, ends this machine's processes for the slug, pushes an `archive-owner/` tag holding its local commits and uncommitted files, and pushes nothing to the branch.
- D2: after an in-loop hand-over no local pair of the slug stands, at once or, for a caller inside the pair, after the parent dobby's next pass;
  a later rejection over the same takeover entry runs no hand-over and pushes nothing for the run.
- D2: a push rejected over another run's commits on a shared records branch replays and pushes, and the run goes on.
- D2: an adopter's takeover push rejected over the old owner's later transition writes the takeover entry onto origin's tip and lands, and the adopter reads the run at that transition.
- D2: after a takeover, a landing past its parent-side merge is driven on exactly one machine, the adopter, and the old owner pushes nothing more to the parent's branch.
- D2: an old owner that merged into its local parent branch and was taken over before pushing it leaves origin's parent branch without the work; the adopter lands the run, origin's parent branch holds the work once, and the old owner's local parent branch is level with its upstream, its merge under an archive tag.
- D2: a Finalize's board `done` reaches origin only after its parent branch push landed.
- D2: a run whose last transition names a code tip origin lacks waits until it lands, then goes on; one still missing after the code-tip grace returns to its previous transition, and the owing machine's later push leaves origin's code branch unchanged;
  a tip given up between the owing machine's read and its push does not stay on origin's code branch, also when the owing machine dies before its re-read, and the owner's next code-side push lands.
- D2: after a takeover, the old owner's next transition and its `lym loom commit-records` put no code commit on origin's code branch beyond the tip the run's last transition on origin names.
- D2: after `lym machine name` on a machine that adopted a run, a rejected records push of that run over its own takeover entry hands nothing over.
- D3: a registration whose former name is another machine's current name is refused naming that machine and `lym machine name --forget-former <name>`; after that verb, the registration lands.
- D3: a registration whose current name is another registered machine's former name, or its current name, is refused naming that machine and `lym machine name <new>`;
  after that verb with a free name, the registration lands, with the refused name among none of this machine's former names.
- D3: an owner recorded as a former name of a registered machine reads as owned here on that machine and as that machine on another, in `dobbyengine.OwnedWorktrees` and `pairteardown.ReadRun` too.
- D3: `lym machine name <new>` puts the old name among the registry's former names on origin when it returns.
- D3: an unregistered machine whose name is a registered machine's current name reads that machine's owners as owned elsewhere, in `dobbyengine.OwnedWorktrees` and `pairteardown.ReadRun` too,
  and its registration is refused naming that machine;
  a machine whose `machine.json` was copied from a registered machine that has since renamed is refused as a copy, naming `lym machine name <new>`.
- D4: a dobby that observes a sleep runs the catch-up before its next pass; a run another machine took meanwhile gets no pass, its processes end, its archive tags reach origin with one notice naming any dropped intent,
  and its local pair is removed with its origin branches left standing.
- D4: a run owned here whose local records are ahead of origin with its own commits has them pushed by the catch-up, and none is reset.
- D4: a run owned elsewhere at `Finalize` past its parent-side merge has its processes end here, and its owner finishes the Finalize.
- D4: with origin unreachable, the catch-up acts on no run and retries; no code path reads or writes the origin-owed marker.
- D4: a hand pass's catch-up fetches only its own run's branches.
- D5: under an `adopt` hold no queued task is claimed and no run but a landing past its parent-side merge gets a pass, on the adopting machine and on another after its Bolt pull; under `adopt-fix` the fix task gets passes, and its agent turns and network calls pass the gate.
- D5: a machine reading a hold holds its runs' processes through `bolt-hold` slug entries, writes no hub entry, and drops only those entries once the hold clears, leaving an operator's hub pause standing.
- D5: under a hold, a run whose push is owed on a machine other than the adopter's gets no `bolt-hold` entry, its owed push lands once origin answers, or is archived by the hand-over when a takeover landed first, and it starts no producer until the hold clears.
- D5: a Finalize cut between its local merge and its parent push leaves origin's parent branch unchanged under a hold, and pushes it at its first pass after the hold clears.
- D7: a dobby whose machine re-adopted a run it had handed over keeps the run's pair.
- D6: a plain `lym adopt` of other subjects under this machine's own hold refuses, naming the hold's subjects.
- D5: a loom or shuttle refusal of a run under a `bolt-hold` entry names the hold's machine and no `lym tender resume`.
- D5: a verb refused by a hold names the hold's machine and a way forward that names no `--take-hold`; `lym loom resume` refuses a run under a hold another machine wrote since the worktree's last Bolt pull.
- D5: a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby on this machine, and no code path writes the `owner` caller.
- D5: at the upgrade's first read a standing `adopt` entry becomes a pause intent and writes one Bolt `adopt` hold with one notice naming it and `lym tender resume` of each run it paused;
  with an open adopt fix task the hold is `adopt-fix` naming it, with the local try count; a standing `owner` entry or a landed run's entry is dropped; an upgrade whose entries all belong to landed runs or runs owned elsewhere writes no hold; a second machine's upgrade under a standing hold writes none.
- D6: `lym adopt <beam>` from a second machine's hub, with origin as the only shared state, ends with the beam's runs in the same FSM states as on the first machine and this machine as owner.
- D6: an adopt on a machine with no local pair of a moved run adds the pair from origin before it writes the takeover entry.
- D6: two machines adopting at once end with one owner; the other's adopt refuses naming the winner.
- D6: a green adopt check leaves no `bolt-hold` pause entry on the adopting machine when the adopt returns.
- D6: a plain `lym adopt` re-run under this machine's own `adopt-fix` hold keeps the hold's kind, fix task and try count, and a failing check files no second task.
- D6: a plain `lym adopt` re-run under this machine's own `adopt-fix` hold leaves the fix task's branches and processes untouched, and its catch-up pushes the owner's owed transitions without resetting them.
- D6: `lym beam status` under a hold shows its kind, machine, time and subjects, under `adopt-fix` its fix task and try counts, and for this machine's own hold the `lym adopt <subjects>` re-run;
  no code path reads the adopt record.
- D6: a failing adopt check files one fix task, leaves an `adopt-fix` hold naming it and exits non-zero; the task's green re-check clears the hold and the runs go on with no hand lift; the try count reads the same on every machine.
- D6: `lym adopt --take-hold` of the hold's subjects on another machine rewrites the hold to name that machine and runs its steps and check; with other subjects it refuses naming the hold's subjects; an agent's shell is denied it, and the row's steer and notice name no `--take-hold`.
- D6: an adopter whose hold another machine took over stops at its next hold re-read, naming that machine, and starts no dobby.
- D6: an adopt killed after its owner write goes on from tracked state on its re-run and ends as an uninterrupted adopt does.
- D6: `lym adopt --here` on the machine that owns the subject ends ok with `owned_here` after one Bolt pull, writes no hold and starts nothing.
- D6: no code path writes the `adopt` or `adopt-hold` caller, and the dobby's handover and the orch watcher read a hold where they read the adopt record.
- D7: a landed run's pair and a cancelled run's pair without keep are torn down by the dobby; one another machine owns or a pause or hold covers is not; a kept pair stays and is listed with its age in `lym beam status`, also after its board entry is removed.
- D7: a pair kept by a cancel before the upgrade, its batten record in shed state `cancelled` with no cancel intent, is listed as kept after the first dobby start and is not torn down.
- D7: an orphan with no commit and no live process is torn down no earlier than the orphan grace after its notice, its branches archived; one with a commit inside the grace stays.
- D7: `lym tender resume` spawns no pair push when no pause deferred one, and still pays a deferred anchor-change push; the hub workspace file no longer lists a pair after its teardown.
- D8: `lym board list` and `lym board get` print `parent_beam` `root` for an entry under the prime and the beam's slug under a beam; `lym board list --text` shows a `P0` row; `hubgeom.ParentOf` still returns the empty slug for the prime.
- Review: D8's readers, `contracts/orchrole/orch-role.md`, `plugins/ly/skills/board/SKILL.md` and the `boardcli` help, read `root` as the prime and address its orch as `<shortname>:orch`.
- D9: `lym beam stop --all` leaves no orch strand, orch Claude process, dobby, coverage upkeep, run process or reed session of the hub on this machine, its own session ended last, and leaves no pause entry of its own and no tracked change.
- D9: under a standing Bolt hold, `lym beam stop --all` ends everything and leaves the hold; under an operator's hub pause it ends everything and leaves that pause standing.
- D9: `lym beam stop <slug>` and the cancel of a beam end the worktree's registered orch Claude process.
- D2: a Finalize stopped between its local merge and its parent push pushes the parent branch at its next pass on the same machine before its transition.
- D9: after `lym beam stop --all` and `lym beam start --all`, a run cut mid-step halts `loop-exited` and its pushed transitions stand.
- D9: hibernate's pause-and-confirm and the stop's run through the same function.
- D10: `lym machine start` starts the prime and owned beams of a hub this machine owns; under a prime another machine owns, or one with no recorded owner, it starts only the beams owned here, names the prime's machine, and claims no prime.
- D10: with a hold standing, `lym machine start` prints the hold's machine and subjects, and for this machine's own hold the `lym adopt <subjects>` re-run.
- D10: a hub's launcher is written at the first start on this machine of a dobby of a worktree this machine owns, a beam under a prime another machine owns included, stays while this machine owns anything in the hub, and is removed when it owns nothing there or the hub leaves `hubregistry.Live`; the all-hubs launcher goes with the last hub.
- D10: where no terminal launcher resolves, `lym machine start` prints each worktree's attach command.

## PATTERN changes

- `PATTERN-run-fsm` — adds: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, both before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first.
- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until an adopt's check is green.

## Notes for the plan writer

- Code dependencies, for the split: D1 before D2; D1 and D2 before D3 to D5 (synchronous Bolt writes, the hand-over); D3 to D5 before D6 (holds, registry, catch-up) and before D7 (hold and owner in the teardown condition);
  D3 before D10 (registry-aware owner filter); D1 before D9 (the stop's bound rests on it).
  D8 shares `boardengine` with D3 and D5 and no code with the rest.
- `PATTERN-owner-machine`'s background gains the one exception to its owner-only pushes:
  a catch-up pushes the owed commits a machine made while it owned a run, until a takeover entry lands (D4 step 4).
- `PATTERN-owner-machine`'s background describes the origin-owed marker and the owner pause; both retire with D4 and D5, and `PATTERN-dobby-runs`' "Adopt pause" cluster retires with D6.
- The two-machine criteria need two hubs over one fake origin with different machine names; `hubforge` builds hub fixtures.
- The study `docs/research/loom/adopt-build-revision-2026-11-16/report.md` has per-package change estimates for D1 to D7.
