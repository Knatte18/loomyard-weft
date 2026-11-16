## Goal

lym beam stop --all takes the hub down on this machine, behind a stop button

Takes: hibernate's pause-and-confirm inline in carry (internal/hibernateengine/watch.go) with awaitConfirmation; beamcli's stop command and its start --all; orchcli.BeamDown and its callers (beam stop, a beam's cancel, pawl's Beam-Down row); runproc.EndSubject and RoleOrch; lym reed down's pair end (internal/reedcli/up.go); tenderengine.Reap and Lift; selfexec.LymBinary and fabricengine's logged detached spawn as the pattern; reed's prime button group in both templates; D1's pushed-or-owed transitions
Delivers: hibernate's pause-and-confirm as one exported function hibernate's carrier and the stop both call, owning only an entry it added; lym beam stop --all re-starting itself detached with a logged spawn and running D9's sequence: hub pause and confirm, each task pair's processes and reed session through reed down's path, orchcli.BeamDown for the prime and each local beam, the beams' reed sessions, unowned run processes as Reap ends them, its own pause entry lifted through Lift, the prime's reed session last; orchcli.BeamDown ending the orch's registered Claude process through runproc as its last step for every caller; a stop entry with a confirm text in reed's prime button group of both templates; no tracked record, no per-run intent, a standing Bolt hold left as it is
Exports: hibernateengine's exported pause-and-confirm function; beamcli's stop --all; BeamDown's added end of the orch's Claude process. It touches no internal/dobbyengine, internal/machinecli or internal/termwindow code, which machine-start-launchers changes beside it

## Scope

The part changes these packages:

- internal/hibernateengine (about 180 changed lines)
- internal/beamcli (about 120 changed lines)
- internal/orchcli (about 70 changed lines)
- internal/reedcli (about 40 changed lines)
- internal/reedengine (about 15 changed lines)

## Decisions

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

- D9: `lym beam stop --all` leaves no orch strand, orch Claude process, dobby, coverage upkeep, run process or reed session of the hub on this machine, its own session ended last, and leaves no pause entry of its own and no tracked change.
- D9: under a standing Bolt hold, `lym beam stop --all` ends everything and leaves the hold; under an operator's hub pause it ends everything and leaves that pause standing.
- D9: `lym beam stop <slug>` and the cancel of a beam end the worktree's registered orch Claude process.
- D9: after `lym beam stop --all` and `lym beam start --all`, a run cut mid-step halts `loop-exited` and its pushed transitions stand.
- D9: hibernate's pause-and-confirm and the stop's run through the same function.
