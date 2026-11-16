## Goal

A machine registry and adopt holds in Bolt, and a catch-up from origin after every absence

Takes: transition-push-takeover's synchronous Bolt writes, takeover reader, hand-over, hand-over mark and owed-push facts, and its pair-push entry taking the set of names reading as this machine; runfsm's Held and OwnedElsewhere inputs, Judge and Aside, and pairteardown.ReadRun; pairteardown.EndSubjectProcesses and its local mode; boardengine's owner setter and OwnedElsewhere; machinename.File, FilePath and Names and lym machine name's owner sweep (internal/machinecli/set.go); dobbyengine's OwnedWorktrees, EnsureLive, loop, handover and PauseOwned seam, its OriginOwed passes; tendercli.PauseOwnedElsewhere; pauserecord's callers, CoveredError and SleepWatch; battencli's origin-owed marker; hibernate's wake hold of runs owned elsewhere; the fix-tries.json reader and writer in internal/adoptcli/holds.go (the unexported readTries and writeTries) and their callers in adoptcli's fix loop, green clear and ResetTries; fabricengine.AdoptScratchDir; dobbyengine's import allowlist (internal/dobbyengine/seam_enforcement_test.go)
Delivers: board.json's machines map and holds list written only by boardengine's setters with a synchronous push; a random machine id in machine.json kept by every writer, forgotten names and lym machine name --forget-former; registration at the first catch-up, dobby start, owner write and lym machine name with every D3 refusal and its way forward; every machine-name comparison through the registry, OwnedElsewhere, OwnedWorktrees, EnsureLive and ReadRun included, and the name set handed to fabricengine's pair push; the catch-up after a sleep, a dobby start or a hand pass, pushing owed commits, handing over runs owned elsewhere and removing their local pairs in local mode; the origin-owed marker retired; ReadRun filling Held, false for the adopt-fix hold's fix task, and the held refusal naming the hold's machine; bolt-hold slug entries under pauserecord.BoltHoldCaller, read by pauserecord's readers and worded by CoveredError; the owner caller and its writers retired into hand_over; the upgrade's first read turning standing adopt and adopt-hold entries into pause intents and one Bolt hold, run by the dobby start before its catch-up in internal/dobbyengine; the fix-tries.json reader and writer moved out of internal/adoptcli/holds.go into internal/fixtries (new) as fixtries.Read and fixtries.Write, importing only internal/state and taking the subject's directory from the caller, so neither internal/dobbyengine nor internal/adoptcli imports the other, adoptcli's callers switched to them and dobbyengine's import allowlist gaining internal/fixtries; the first read calling fixtries.Read on each subject's directory under the adopt scratch directory its caller hands in
Exports: boardengine's registry and hold types, their readers, and the one hold write the upgrade's first read calls, writing an adopt or adopt-fix hold; OwnedElsewhere taking the registry; the catch-up entry point the dobby start calls here and adopt calls later, run by the dobby start in internal/dobbyengine; pauserecord.BoltHoldCaller; fixtries.Read, called by the upgrade's first read and adoptcli's fix loop, and fixtries.Write, called by adoptcli's fix loop, green clear and ResetTries

## Scope

The part changes these packages:

- internal/boardengine (about 450 changed lines)
- internal/dobbyengine (about 450 changed lines)
- internal/machinecli (about 300 changed lines)
- internal/pauserecord (about 160 changed lines)
- internal/battencli (about 140 changed lines)
- internal/orchcli (about 120 changed lines)
- internal/machinename (about 100 changed lines)
- internal/tenderengine (about 100 changed lines)
- internal/beamcli (about 100 changed lines)
- internal/pairteardown (about 60 changed lines)
- internal/tendercli (about 60 changed lines)
- internal/hibernateengine (about 60 changed lines)
- internal/runfsm (about 50 changed lines)
- internal/adoptcli (about 40 changed lines)
- internal/fixtries (about 40 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/battenshed (about 30 changed lines)
- internal/hubgeom (about 20 changed lines)
- internal/landingshed (about 10 changed lines)
- internal/fabricengine (about 10 changed lines)
- internal/shedrecipe (about 10 changed lines)
- internal/shedrun (about 10 changed lines)
- internal/hibernatecli (about 10 changed lines)
- internal/fabriccli (about 10 changed lines)
- internal/fronttask (about 10 changed lines)
- internal/pawlengine (about 10 changed lines)
- pattern (about 10 changed lines)

## Decisions

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

## Acceptance criteria

- D1: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns.
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
- D5: a loom or shuttle refusal of a run under a `bolt-hold` entry names the hold's machine and no `lym tender resume`.
- D5: a verb refused by a hold names the hold's machine and a way forward that names no `--take-hold`; `lym loom resume` refuses a run under a hold another machine wrote since the worktree's last Bolt pull.
- D5: a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby on this machine, and no code path writes the `owner` caller.
- D5: at the upgrade's first read a standing `adopt` entry becomes a pause intent and writes one Bolt `adopt` hold with one notice naming it and `lym tender resume` of each run it paused;
  with an open adopt fix task the hold is `adopt-fix` naming it, with the local try count; a standing `owner` entry or a landed run's entry is dropped; an upgrade whose entries all belong to landed runs or runs owned elsewhere writes no hold; a second machine's upgrade under a standing hold writes none.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first.
