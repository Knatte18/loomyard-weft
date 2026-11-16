## Goal

The dobby tears down or reports leftover pairs

Takes: runfsm's landed and cancelled states and runfsm.Derive's reading of a cancel intent with its finish; dobbyengine's ended-run teardown (strandedEnded in internal/dobbyengine/ended.go); pairteardown's teardown and local mode; the hand-over mark from transition-push-takeover and its drop at adopt's takeover write from adopt-foreground; the holds, registry and owner reads from machine-registry-holds-catchup; D1's synchronous push for the kept-cancel intent; lym tender resume's pair push; fabricengine.RegenerateHubWorkspace
Delivers: the dobby tearing down landed pairs and cancelled pairs without keep, by FSM state, with no pause, hold or other owner covering them, a cancelled run's batten record going in the same step; the dobby removing in local mode the pair of a run its hand-over mark names while another machine owns it; kept pairs listed with their age in lym beam status and in one notice per dobby start naming lym fabric remove <slug>; a finished keep cancel intent for each pre-upgrade kept-cancel record at the first dobby start; orphans noticed, their first-sight time in a machine-local file named in internal/lymdirs, and torn down idle after the orphan grace with their branches archived; lym tender resume spawning no pair push for a transition; the hub workspace file regenerated after each teardown
Exports: the orphan grace constant and the first-sight file's path

## Scope

The part changes these packages:

- internal/dobbyengine (about 250 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/pairteardown (about 60 changed lines)
- internal/fabricengine (about 50 changed lines)
- internal/battencli (about 40 changed lines)
- internal/tendercli (about 30 changed lines)
- internal/shedrun (about 20 changed lines)
- internal/lymdirs (about 10 changed lines)

## Decisions

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

- D2: after an in-loop hand-over no local pair of the slug stands, at once or, for a caller inside the pair, after the parent dobby's next pass;
  a later rejection over the same takeover entry runs no hand-over and pushes nothing for the run.
- D7: a dobby whose machine re-adopted a run it had handed over keeps the run's pair.
- D7: a landed run's pair and a cancelled run's pair without keep are torn down by the dobby; one another machine owns or a pause or hold covers is not; a kept pair stays and is listed with its age in `lym beam status`, also after its board entry is removed.
- D7: a pair kept by a cancel before the upgrade, its batten record in shed state `cancelled` with no cancel intent, is listed as kept after the first dobby start and is not torn down.
- D7: an orphan with no commit and no live process is torn down no earlier than the orphan grace after its notice, its branches archived; one with a commit inside the grace stays.
- D7: `lym tender resume` spawns no pair push when no pause deferred one, and still pays a deferred anchor-change push; the hub workspace file no longer lists a pair after its teardown.
