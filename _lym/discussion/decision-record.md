## Goal

lym adopt runs in the foreground over pushed state: hold, owner, pull, takeover, check, with --take-hold

Takes: machine-registry-holds-catchup's hold types and readers, the hold write its upgrade first read calls, its registry types, readers and registration write with D3's refusals, its catch-up entry point, pauserecord.BoltHoldCaller, and fixtries.Read and fixtries.Write in internal/fixtries, which its upgrade first read and adoptcli's fix loop call; boardengine.SetOwners's pull-first commit and its re-read after a rejected push (internal/boardengine/ownerset.go); transition-push-takeover's takeover entry type, reader and synchronous pushes, and the hand-over mark's path and reader; runfsm's states and the refusals that name lym adopt <subject>; adoptcli's fix loop (fixloop.go, fronttask.File), check, deploy, archives and --here paths; the adopt record's readers in the dobby's handover, the orch watcher and lym beam status; orchcli.BeamUp; internal/shellguard's table
Delivers: lym adopt running hold and owner write, pull, takeover, catch-up, deploy, workspace, check and green clear in the caller's process, re-runnable from tracked state, with the same-set and other-set rules of this machine's own hold; boardengine's hold setters beside their callers here: rewrite a hold to name another machine keeping its kind, fix task, try counts and owners before the adopt, replace it with an adopt-fix hold, update try counts and clear it; one pull-first Bolt write in boardengine recording the adopt hold, or under --take-hold the rewritten hold, together with the subjects' owner and this machine's registry entry in one commit, which on a rejected push re-reads Bolt and refuses on an owner of these subjects or a hold another machine wrote since; the takeover entry's production writer and its rejected-push retry, step 3's takeover write dropping this machine's hand-over marks of each moved slug through a drop function added beside the mark reader; lym adopt --take-hold denied to agents by a shell-guard row whose steer and notice never spell the flag; lym adopt check reading the adopt-fix hold's try counts; --here keeping no_subject and owned_here; the worker, the adopt record and its readers, the held-back project verdicts, the adopt pause step, the adopt and adopt-hold caller writers and --keep-local, --drop-local and --discard retired; the dobby's handover and the orch watcher reading a hold; lym beam status showing the Bolt hold; adoptcli's and adoptrecord's package docs rewritten or the package removed, fixtries.Read in internal/fixtries staying for the upgrade's first read
Exports: adopt's foreground step runner, the takeover entry's production writer, and boardengine's hold setters and one-commit adopt write, each with its production caller in adopt

## Scope

The part changes these packages:

- internal/adoptcli (about 2650 changed lines)
- internal/adoptrecord (about 400 changed lines)
- internal/boardengine (about 250 changed lines)
- internal/dobbyengine (about 150 changed lines)
- internal/fabricengine (about 120 changed lines)
- internal/beamcli (about 80 changed lines)
- internal/orchcli (about 80 changed lines)
- internal/pauserecord (about 80 changed lines)
- internal/battencli (about 60 changed lines)
- internal/shellguard (about 30 changed lines)
- cmd/lym (about 10 changed lines)
- pattern (about 10 changed lines)

## Decisions

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

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/plan/round-3-review.md; fixer report: _lym/reviews/plan/round-3-fixer-report.md.
  - takeover-teardown-lock-refusal-unreachable (class design, severity MEDIUM, open in rounds 3)
<!-- /lym:carry-over Plan-Review -->

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - fixhold-way-forward-names-only-failing-subject (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D2: an adopter's takeover push rejected over the old owner's later transition writes the takeover entry onto origin's tip and lands, and the adopter reads the run at that transition.
- D6: a plain `lym adopt` of other subjects under this machine's own hold refuses, naming the hold's subjects.
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

## PATTERN changes

- `PATTERN-dobby-runs` — changes: Starting or holding a board task's run: only its parent's dobby, started with its orch bar `lym adopt --no-orch`; front tasks only via `fronttask`; an adopt holds the hub through Bolt's holds until an adopt's check is green.
