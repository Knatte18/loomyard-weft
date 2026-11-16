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
  A decision below fills or calls them and never redesigns them.
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
- A transition takes a network gate admission (`pauserecord.Admit`) before its commit and releases it once every side has landed or its push turns owed,
  so a pause or hold written after the commit stops the next producer and never that push's first attempt.
  Fabric's pause deferral no longer applies to a transition's push; it stays for the detached pushes below.
- The anchor change's push (`internal/fabricengine/anchorchange.go`) and tender's prime push (`internal/tendercli/deps.go`) stay detached:
  neither carries a run transition.
- A push that fails for a transient reason, such as an unreachable origin, leaves the run waiting at that transition with the push owed.
  It retries on a backoff and waits at the gate like any network call, so a pause holds it.
  `pairteardown.ReadRun` reads the owed push into `runfsm.Inputs`' `PushOwedOffline`, which nothing fills today,
  and `lym beam status` shows the owed push and its last error; the failed-push record and its creel notice stay.
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
  When it holds a takeover entry naming another machine that this machine has not yet read, the run hands over without replaying:
  - it pushes no code side for the rejected push;
  - it ends its processes for the slug through `pairteardown.EndSubjectProcesses`, with the dobby end unset when the caller is that dobby's own run;
  - it pushes the code tip the run's last transition on origin names when origin's code branch lacks it, since that transition stands;
  - it archives the local tips as `fabricengine.ArchiveOwnerTips` does, uncommitted files first committed onto the archived tip,
    and its local parent branch's tip when this run's parent-side merge alone puts that branch ahead of its upstream, then resets that branch to its upstream;
  - it pushes the archive tags and posts one notice naming the machine and any pause or cancel intent it dropped.
  It pushes nothing more for the run.
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
  A machine owing a tip pushes it only while the run's last transition on origin names it, so a given-up tip never lands.
- A rejection whose fetched record holds no takeover entry, such as one over other runs' commits on a shared records branch like `main-weft`, replays and pushes as today.
- An adopter's own takeover push that is rejected fetches, then re-reads its hold and the subjects' owner from a fetched Bolt.
  When its hold is gone or either names another machine, it stops, naming the subjects' owner.
  Otherwise it writes the takeover entry again onto origin's tip and pushes again; it never replays its own commit and never hands over.

Bound: transitions the old owner pushed before the takeover landed stand, and the adopter goes on from them.
The old owner's step in flight runs to its next transition, which the takeover rejects; its result is archived under a tag.
A pause or cancel intent owed on the old owner when the takeover lands is dropped, and the notice names it to repeat on the owner machine.

Rationale: a takeover entry in the run's own record is a compare-and-swap per run that holds on records branches many runs share, and costs no extra fetch per transition.

### D3 — A machine registry in Bolt gives each machine one identity

- `board.json` gains a top-level `machines` map: each registered name with the former names it replaced.
  Only `boardengine`'s owner setter writes it, pushing synchronously.
- A machine registers at its first owner write or `lym machine name`, copying the former names of its `machine.json`.
  A registration whose former name is another registered machine's current name is refused, naming that machine;
  its way forward is `lym machine name --forget-former <name>` (new) on this machine, which drops that former name from its `machine.json` and its registry entry.
- `lym machine name` records the rename in the registry, besides its sweep of owners (`internal/machinecli/set.go`).
- Every comparison of a recorded machine name with this machine goes through the registry: a former name of a registered machine reads as that machine.
  `boardengine.OwnedElsewhere` takes the registry, every owner write writes the current name,
  and the comparisons that name the machine directly move too: `dobbyengine.OwnedWorktrees` and `dobbyengine.EnsureLive`.
- `internal/machinename` stays this machine's only name source.

Bound: `--forget-former` changes only this machine's own file and entry; a registration clash refuses and never overwrites another machine's entry.

Rationale: `osl46` and `osl-1046` are one machine before and after `lym machine name`, and only that machine's `machine.json` knows it.

### D4 — After an absence, lym catches up from origin before it acts

An absence is a sleep a long-lived lym process observes through `pauserecord.SleepWatch` over `pauserecord.SleepTolerance` (the dobby loop or the orch watcher),
a worktree's dobby starting, or a hand pass, whose catch-up runs over its own run only.

Before any act rule answer after an absence, the catch-up runs:

1. It fetches Bolt, replays local display commits as the board sync does, and reads the owners, the registry and the holds.
2. It fetches each local pair's branches and reads each run's record from the result.
3. For a run this machine owns, local commits on top of origin's tip are owed pushes, and it pushes them; a takeover entry naming another machine is D2's hand-over.
4. For a run another machine owns, a `landing` past its parent-side merge included, it ends this machine's processes for the slug through `pairteardown.EndSubjectProcesses`,
   pushes the code tip the run's last transition on origin names when origin lacks it, archives the local tips as D2 does,
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
  A hold has a kind, `adopt` or `adopt-fix`, the machine, the time, its adopt's subjects, and for `adopt-fix` the fix task's slug and each subject's try count.
  Only `boardengine`'s setter writes it, pushing synchronously.
- A hold covers the whole hub: the act rule answers `wait` for every run on every machine.
  `pairteardown.ReadRun` sets `runfsm.Inputs`' `Held`, false for the `adopt-fix` hold's fix task; the act rule already lets a `landing` past its parent-side merge act under a hold.
  The refusal of a held run names the hold's machine, so `runfsm` carries it in its inputs and verdict;
  its way forward is waiting for the hold to clear, or asking the operator when that machine has stopped.
- Every verb that asks the act rule outside the dobby's pass, such as `lym loom resume` and `lym tender resume <slug>`, fetches Bolt first when origin answers.
- A machine that reads a hold writes no hub entry.
  It writes a slug entry under one new `pauserecord` caller `hold` for each of its runs the act rule answers `wait` for under the hold,
  none for the `adopt-fix` hold's fix task, and drops those entries once it reads no hold.
- The `hold` entries hold only those runs: the orch, `lym adopt`'s own process and its deploy and check children, and the operator's `lym batten cancel` go on.
  The orch is up while a hold stands (operator, 2026-11-15).
- The `hold` entries keep apart from the operator's hub entry and hibernate's: dropping them drops only the `hold` caller's,
  and the hibernate wake never reads a `hold` slug entry as a hub entry.
  A slug with a live entry gets no `hold` entry.
  `lym tender resume` lifts no `hold` entry while the Bolt hold stands; lifting a slug's own entry under a standing hold writes the `hold` entry in its place.
  `lym tender resume <slug>` of a slug only a `hold` entry covers refuses, naming the hold's machine and the way forward above.
- A run another machine owns is not paused: the act rule answers `hand_over`, and `pairteardown.EndSubjectProcesses` ends its processes.
  The `owner` caller retires with every writer: the dobby's owner pause (`internal/dobbyengine/owned.go`), `tendercli.PauseOwnedElsewhere` and the process end beside it,
  and the hibernate wake's hold of runs owned elsewhere.
  The `adopt` and legacy `adopt-hold` callers' writers retire with adopt's pause step (D6); the two names stay declared only for the upgrade's first read.
- At the first read after the upgrade, a standing `adopt` or `adopt-hold` entry of a run this machine owns in a state that is not terminal becomes a run pause intent, so no run resumes unasked.
  Every other entry of those callers, and every `owner` entry, is dropped.
  When at least one entry became a pause intent or belonged to a released `queued` task, the first read writes one Bolt `adopt` hold naming this machine and the subjects of those runs and tasks,
  and posts one notice naming the hold, its machine and, as way forward, a re-run of `lym adopt` of those subjects there.
  A first read that finds another machine's hold writes none, and its notice names that hold's machine.
- The dobby's handover and the orch watcher, which ask the adopt record whether an adopt here covers a worktree, read a hold this machine wrote instead, once D6 retires the record.

Bound: a machine other than the adopter learns of a hold at its next Bolt pull or catch-up; until then its dobby goes on for at most the dobby's pull interval, and D2's bound covers the transitions it pushes.
A run holding its parent lock gets no slug entry, so the gate does not hold its step in flight; the act rule still starts no producer for it.
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
2. For each moved run it fetches the run's branches, archives local tips origin lacks as D2 does, resets them to origin's, and adds the pair from origin when this machine has none.
3. It writes the takeover entry on each moved run's records branches and pushes it (D2), then fast-forwards the local records branches and code branch to origin's tips.
4. The catch-up (D4) over the whole hub.
5. The deploy, as today.
6. The hub workspace file regenerated.
7. The check: the hub's `adopt` gate in each subject's worktree, in the foreground, its output on the terminal.
8. A green check clears the hold, drops this machine's `hold` pause entries at once, and runs `orchcli.BeamUp` for each subject; the runs go on by the act rule.
   A failing check files a fix task through the fix loop (`internal/adoptcli/fixloop.go`, `fronttask.File`), replaces the hold with an `adopt-fix` hold naming it,
   runs `orchcli.BeamUp`, and exits non-zero naming the task.
   The task's landing runs the check again: green clears the hold, failing files the next task within `adopt_fix_tries`.
   Each subject's try count lives in the `adopt-fix` hold, so a re-check on any machine reads the same count.
   At the limit the hold stays, and the dobby's notice names the failing tests and the fix task.

`lym adopt --take-hold <subject>...` (new) takes over a hold another machine wrote.
It refuses a hold that names other subjects, naming the hold's subjects.
Otherwise its Bolt commit rewrites the hold to name this machine, keeping kind, fix task and try counts, writes this machine as owner, pushes, and runs steps 2 to 8.
Under `adopt-fix` the fix task stands and a failing check files no second task.
Every adopt re-reads its own hold from Bolt before each step from step 2 on, and stops once the hold is gone or names another machine, naming the subjects' owner and starting nothing.
A shell-guard row denies `--take-hold` to every agent; adopt's help names it.

Kept: `--archives`, `--no-orch`, `--all` and `--here`.
`--here` stays behind `lym s` and keeps its fast outcomes: `no_subject` in a worktree with no subject,
and `owned_here` after one Bolt pull when no other machine owns the subject, writing no hold and starting nothing.
Retired: `--keep-local`, `--drop-local`, `--discard`, `lym adopt worker`, the adopt record, the held-back project verdicts, the adopt pause step and every writer of the `adopt` and `adopt-hold` callers.
`internal/adoptcli`'s and `internal/adoptrecord`'s package docs are rewritten to the foreground flow, or the package removed with the record.

Bound: adopt works from pushed state alone; the old owner's unpushed work reaches the adopter only through D2's archive tags, which `--archives` lists.
`--take-hold` skips no check and lets no run go on before a green check: it moves the hold and its adopt to this machine, and only the operator's own shell runs it.

Rationale: the operator decided adopt runs in the foreground and that nothing happens in the hub until it is done and every failing test it found is fixed;
D1 and D2 leave no local state for adopt to reconcile.

### D7 — Leftover pairs are torn down or reported

- The dobby tears down, through `internal/pairteardown`, the pair of each run in the FSM state `landed`, or `cancelled` without keep,
  with no pause or hold covering it and no other machine owning it; a cancelled run's batten record goes in the same step.
  This widens the ended-run teardown of `internal/dobbyengine/ended.go`, whose stranded-run condition reads a board entry that ended and so takes neither case today;
  the widened condition reads the FSM state.
- A cancelled run whose pair was kept stays: `lym beam status` lists it with its age, and each dobby start posts one notice listing kept pairs and naming `lym fabric remove <slug>`.
- A pair kept by a cancel before the upgrade has no batten record.
  The first dobby start after the upgrade writes, for each pair whose slug has a board entry with an empty status, no batten record and no live process,
  a batten record holding a finished cancel with keep.
  An entry with the status `done` gets none, so a landed run's leftover pair reads `landed` and is torn down.
- A pair with neither a board entry nor a batten record is an orphan.
  The dobby posts a notice at first sight saying when it goes, and tears it down no earlier than the orphan grace after that notice,
  only when neither branch has had a commit for the orphan grace and no process of its slug lives; uncommitted files are committed onto the archived tip first.
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

Bound: the stop ends agent steps mid-step; D1 leaves every transition on origin, so nothing pushed is lost and the cut step runs again.
It writes no pause intent and no tracked record, so no other machine reads anything from it, and its own pause entry is gone when it ends.
A stop that dies midway leaves its pause entry; `lym tender resume` lifts it.
The operator closes terminal windows by hand.

Rationale: the operator stops a machine today by `lym beam stop` per worktree, `lym reed down` per pair and `lym tender reap`.
The entry builds the stop on the run intents so that it leaves every run in a pushed state; D1 gives that state, and a per-run intent would be pushed and read as `paused` on other machines (Open risks).
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
  lym adds a hub's launcher, and the all-hubs one with the first hub, when this machine's prime dobby starts.
  It removes a hub's launcher when this machine's verb, dobby or catch-up (D4) finds that this machine owns nothing in the hub,
  and the verb removes each launcher on disk whose hub `hubregistry.Live` no longer lists; the all-hubs one goes with the last hub.
  Beams need no launcher change.

Bound: the verb starts only what this machine owns, and never claims a prime; it writes no tracked record.
A launcher's removal waits until this machine next runs the verb or a dobby, since no other machine reaches this desktop.

Rationale: after a crash the operator finds the hubs and beams that were in flight by hand, and starting them is what the orchs and dobbies cannot do for themselves.

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
- `PATTERN-shell-guard-table`: the deny of `lym adopt --take-hold` is a row of `internal/shellguard`'s table.
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

- The entry's hub-stop-button section (operator, 2026-11-15) builds the stop "on the run FSM and its pause and cancel intents, so a stop leaves every run in a pushed state".
  D9 writes no intent and rests the pushed state on D1 instead, since a pushed intent would read as `paused` on every machine and need origin.
  The parent confirms the operator accepts the stop without per-run intents.

## Acceptance criteria

- D1: a shed transition in a task pair pushes its records side, then its code side once the records side landed, and starts the next producer only after both landed on origin.
- D1: a pause written between a transition's commit and its push leaves that transition on origin, and the run's next producer does not start.
- D1: a transition's push that waits on a held teardown lock past the push lock wait turns owed, and a pause written then holds the process's network calls.
- D1: with origin unreachable, a transition leaves the run waiting with the push owed, starts no producer, shows in `lym beam status`, and goes on once origin answers.
- D1: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns.
- D1: `lym tender pause <slug>`, `lym tender resume <slug>` and `lym batten cancel` return with their intent on origin, and a pull-request record write and a fixer attempt count are on origin before the next producer starts.
- D1: with origin unreachable, `lym tender pause <slug>` holds the run's processes here at once, and the pause intent is on origin once origin answers.
- D2: a records push rejected over a takeover entry naming another machine runs no replay, ends this machine's processes for the slug, pushes an `archive-owner/` tag holding its local commits and uncommitted files, and pushes nothing to the branch.
- D2: a push rejected over another run's commits on a shared records branch replays and pushes, and the run goes on.
- D2: an adopter's takeover push rejected over the old owner's later transition writes the takeover entry onto origin's tip and lands, and the adopter reads the run at that transition.
- D2: after a takeover, a landing past its parent-side merge is driven on exactly one machine, the adopter, and the old owner pushes nothing more to the parent's branch.
- D2: an old owner that merged into its local parent branch and was taken over before pushing it leaves origin's parent branch without the work; the adopter lands the run, origin's parent branch holds the work once, and the old owner's local parent branch is level with its upstream, its merge under an archive tag.
- D2: a Finalize's board `done` reaches origin only after its parent branch push landed.
- D2: a run whose last transition names a code tip origin lacks waits until it lands, then goes on; one still missing after the code-tip grace returns to its previous transition, and the owing machine's later push leaves origin's code branch unchanged.
- D2: after a takeover, the old owner's next transition and its `lym loom commit-records` leave origin's code branch unchanged.
- D3: a registration whose former name is another machine's current name is refused naming that machine and `lym machine name --forget-former <name>`; after that verb, the registration lands.
- D3: an owner recorded as a former name of a registered machine reads as owned here on that machine and as that machine on another, in `dobbyengine.OwnedWorktrees` and `pairteardown.ReadRun` too.
- D3: `lym machine name <new>` puts the old name among the registry's former names on origin when it returns.
- D4: a dobby that observes a sleep runs the catch-up before its next pass; a run another machine took meanwhile gets no pass, its processes end, and its local pair is removed with its origin branches left standing.
- D4: a run owned here whose local records are ahead of origin with its own commits has them pushed by the catch-up, and none is reset.
- D4: a run owned elsewhere at `Finalize` past its parent-side merge has its processes end here, and its owner finishes the Finalize.
- D4: with origin unreachable, the catch-up acts on no run and retries; no code path reads or writes the origin-owed marker.
- D4: a hand pass's catch-up fetches only its own run's branches.
- D5: under an `adopt` hold no queued task is claimed and no run but a landing past its parent-side merge gets a pass, on the adopting machine and on another after its Bolt pull; under `adopt-fix` the fix task gets passes, and its agent turns and network calls pass the gate.
- D5: a machine reading a hold holds its runs' processes through `hold` slug entries, writes no hub entry, and drops only those entries once the hold clears, leaving an operator's hub pause standing.
- D5: a verb refused by a hold names the hold's machine and a way forward that names no `--take-hold`; `lym loom resume` refuses a run under a hold another machine wrote since the worktree's last Bolt pull.
- D5: a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby on this machine, and no code path writes the `owner` caller.
- D5: at the upgrade's first read a standing `adopt` entry becomes a pause intent and writes one Bolt `adopt` hold with one notice naming it; a standing `owner` entry or a landed run's entry is dropped; an upgrade whose entries all belong to landed runs or runs owned elsewhere writes no hold; a second machine's upgrade under a standing hold writes none.
- D6: `lym adopt <beam>` from a second machine's hub, with origin as the only shared state, ends with the beam's runs in the same FSM states as on the first machine and this machine as owner.
- D6: an adopt on a machine with no local pair of a moved run adds the pair from origin before it writes the takeover entry.
- D6: two machines adopting at once end with one owner; the other's adopt refuses naming the winner.
- D6: a green adopt check leaves no `hold` pause entry on the adopting machine when the adopt returns.
- D6: a failing adopt check files one fix task, leaves an `adopt-fix` hold naming it and exits non-zero; the task's green re-check clears the hold and the runs go on with no hand lift; the try count reads the same on every machine.
- D6: `lym adopt --take-hold` of the hold's subjects on another machine rewrites the hold to name that machine and runs its steps and check; with other subjects it refuses naming the hold's subjects; an agent's shell is denied it.
- D6: an adopter whose hold another machine took over stops at its next hold re-read, naming that machine, and starts no dobby.
- D6: an adopt killed after its owner write goes on from tracked state on its re-run and ends as an uninterrupted adopt does.
- D6: `lym adopt --here` on the machine that owns the subject ends ok with `owned_here` after one Bolt pull, writes no hold and starts nothing.
- D6: no code path writes the `adopt` or `adopt-hold` caller, and the dobby's handover and the orch watcher read a hold where they read the adopt record.
- D7: a landed run's pair and a cancelled run's pair without keep are torn down by the dobby; one another machine owns or a pause or hold covers is not; a kept pair stays and is listed with its age in `lym beam status`, also after its board entry is removed.
- D7: a pair kept by a cancel before the upgrade is listed as kept after the first dobby start; a pair whose claim stands, or whose entry is `done`, is not.
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
- D10: a hub's launcher is written at its prime dobby's first start on this machine, stays while this machine owns anything in the hub, and is removed when it owns nothing there or the hub leaves `hubregistry.Live`; the all-hubs launcher goes with the last hub.
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
- `PATTERN-owner-machine`'s background describes the origin-owed marker and the owner pause; both retire with D4 and D5, and `PATTERN-dobby-runs`' "Adopt pause" cluster retires with D6.
- The two-machine criteria need two hubs over one fake origin with different machine names; `hubforge` builds hub fixtures.
- The study `docs/research/loom/adopt-build-revision-2026-11-16/report.md` has per-package change estimates for D1 to D7.
