## Goal

A transition pushes its records before the run acts, and a takeover entry in the run's records hands the run over

Takes: runfsm's Judge, Aside and Inputs, whose PushOwedOffline and CodeTipMissing nothing fills; pairteardown.ReadRun, the one reader of a run's inputs, and EndSubjectProcesses; the detached commit-and-push of lym tender pause, lym tender resume, lym batten cancel, the pull-request record and the fixer attempt counts; fabricengine's PushPairAnchored (code side first today), coalescePushSides, coalesceFlagged and respawnForeignFlags, the teardown-lock respawn, pushclassified's rejection path, anchorchange's push and ArchiveOwnerTips; statuscommit's per-transition spawn; boardengine's Sync, SpawnSync and owner commit; landingshed's Finalize; machinename.Names; PATTERN-test-only-code holds, so every new function has a production caller in this part
Delivers: a shed transition that takes a pauserecord.Admit admission, commits its records naming its code tip and pushes records side first and code side after before the next producer starts; PushPairAnchored and coalescePushSides sending records first and no code side after a failed or rejected records side; the run intents, the PR record and the fixer counts on the same synchronous push, the offline verbs acting here at once and owing their push; the per-transition detached push, the teardown-lock respawn and the pending-flag respawn retired; an owed push retried on a backoff behind the gate and read by ReadRun into runfsm's PushOwed (renamed from PushOwedOffline) with its last error carried in the inputs, the verdict, the refusal and lym beam status; synchronous Bolt writes for a run claim, a landing's done and an owner; Finalize pushing the parent branch before its transition and its board done, and reading the owner before its merge and each later push; a rejected records push that reads origin's record against the name set its caller hands in and hands over on another machine's takeover entry, archiving local tips and the local parent merge, removing the local pair through pairteardown's local mode when the caller runs outside the pair, writing the hand-over mark under the hub's .lym and skipping a takeover a mark names; ReadRun setting CodeTipMissing, the code-tip wait, its 30-minute grace and the leased push and reset of an owed tip
Exports: the takeover entry type and its reader, called in the rejected-push path, and its test writer in internal/testkit; a pair-push entry that takes the set of names reading as this machine, so the registry part passes its names in later without changing fabricengine's signature again; the hand-over mark's path in internal/lymdirs and its reader; runfsm's PushOwed input with its last error; the code-tip grace constant

## Scope

The part changes these packages:

- internal/fabricengine (about 1340 changed lines)
- internal/battencli (about 300 changed lines)
- internal/landingshed (about 250 changed lines)
- internal/sheddriver (about 200 changed lines)
- internal/loomcli (about 200 changed lines)
- internal/boardengine (about 150 changed lines)
- internal/statuscommit (about 100 changed lines)
- internal/tendercli (about 100 changed lines)
- internal/pairteardown (about 100 changed lines)
- internal/runfsm (about 60 changed lines)
- internal/testkit (about 60 changed lines)
- internal/shedengine (about 60 changed lines)
- internal/beamcli (about 60 changed lines)
- internal/fabriccli (about 40 changed lines)
- internal/tenderengine (about 40 changed lines)
- internal/pauserecord (about 40 changed lines)
- internal/pullrequest (about 30 changed lines)
- internal/lymdirs (about 10 changed lines)
- pattern (about 30 changed lines)

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

### Added after Discussion (parent, 2026-11-16): The owner's guard resets a code tip off the run's last transition's line

Decision: D2's owner guard is narrowed: when the owner's code-side push is rejected, it resets origin's code branch with a lease at that tip, then pushes, when origin's tip is neither the tip the run's last transition on origin names nor an ancestor of it. A tip named only by an older transition no longer counts as named.

Rationale: A given-up tip stays named by the transition the give-up returned from, so D2's text (a tip that no transition of the run on origin names) would never reset it and would leave the given-up tip on origin, against D2's own aim. An ancestor of the last named tip is origin lagging behind, which the owner's push moves forward without a reset.

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

- D1: a shed transition in a task pair pushes its records side, then its code side once the records side landed, and starts the next producer only after both landed on origin.
- D1: a pause written between a transition's commit and its push leaves that transition on origin, and the run's next producer does not start.
- D1: a transition's push that waits on a held teardown lock past the push lock wait turns owed, and a pause written then holds the process's network calls.
- D1: with origin unreachable, a transition leaves the run waiting with the push owed, starts no producer, shows in `lym beam status`, and goes on once origin answers.
- D1: a push owed on a held teardown lock is refused naming the owed push and its last error, not an unreachable origin.
- D1: a code side flagged by a push whose lock wait gave up never reaches origin before its records side, nor after a takeover entry naming another machine.
- D1: an anchor change's push in a pair another machine took over leaves origin's code branch unchanged.
- D1: `lym tender pause <slug>`, `lym tender resume <slug>` and `lym batten cancel` return with their intent on origin, and a pull-request record write and a fixer attempt count are on origin before the next producer starts.
- D1: with origin unreachable, `lym tender pause <slug>` holds the run's processes here at once, and the pause intent is on origin once origin answers.
- D2: a records push rejected over a takeover entry naming another machine runs no replay, ends this machine's processes for the slug, pushes an `archive-owner/` tag holding its local commits and uncommitted files, and pushes nothing to the branch.
- D2: a push rejected over another run's commits on a shared records branch replays and pushes, and the run goes on.
- D2: after a takeover, a landing past its parent-side merge is driven on exactly one machine, the adopter, and the old owner pushes nothing more to the parent's branch.
- D2: an old owner that merged into its local parent branch and was taken over before pushing it leaves origin's parent branch without the work; the adopter lands the run, origin's parent branch holds the work once, and the old owner's local parent branch is level with its upstream, its merge under an archive tag.
- D2: a Finalize's board `done` reaches origin only after its parent branch push landed.
- D2: a run whose last transition names a code tip origin lacks waits until it lands, then goes on; one still missing after the code-tip grace returns to its previous transition, and the owing machine's later push leaves origin's code branch unchanged;
  a tip given up between the owing machine's read and its push does not stay on origin's code branch, also when the owing machine dies before its re-read, and the owner's next code-side push lands.
- D2: after a takeover, the old owner's next transition and its `lym loom commit-records` put no code commit on origin's code branch beyond the tip the run's last transition on origin names.
- D2: after `lym machine name` on a machine that adopted a run, a rejected records push of that run over its own takeover entry hands nothing over.
- D2: a Finalize stopped between its local merge and its parent push pushes the parent branch at its next pass on the same machine before its transition.

## PATTERN changes

- `PATTERN-run-fsm` — adds: Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays.
- `PATTERN-pair-remote` — changes: Pushing run records or tearing down a pair: the code branch goes along after the records side, both before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`.
