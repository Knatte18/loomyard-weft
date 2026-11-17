## Goal

lym machine start brings back every hub this machine owns, from desktop launchers

Takes: machinecli's cobra subtree and deps seams; hubregistry.Live and Lookup; dobbyengine.OwnedWorktrees and EnsureLive with their registry-aware owner reads and the catch-up from machine-registry-holds-catchup, run at the dobby start, after a sleep and on a hand pass; the Bolt hold reader; lym beam start --all as a child process named through selfexec.LymBinary; termwindow.Resolve and OpenAttach; fabricengine.HubWorkspacePath and vscode.Launch
Delivers: lym machine start [<shortname>] starting the prime and owned beams of one hub or every live hub through lym beam start in the hub's prime, skipping a prime another machine owns or one with no recorded owner and saying so, never through lym s; the hub's workspace file opened and one terminal per started worktree running lym reed attach, or each attach command printed where no launcher resolves; a standing hold printed with its machine, subjects and, for this machine's own, the lym adopt <subjects> re-run; native Windows (wt.exe) in termwindow; internal/desktoplauncher writing .desktop files on Linux and .cmd files on the Windows desktop, through wsl.exe under WSL; the launcher added at the dobby start and removed by every catch-up, the dobby start's included, that finds this machine owns nothing in the hub, in internal/dobbyengine, and added and removed by the verb, never in internal/beamcli
Exports: internal/desktoplauncher's add and remove functions and machinecli's start verb

## Scope

The part changes these packages:

- internal/machinecli (about 420 changed lines)
- internal/desktoplauncher (about 380 changed lines)
- internal/termwindow (about 90 changed lines)
- internal/dobbyengine (about 80 changed lines)

## Decisions

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

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - no-prime-claim-flag-literal-repeated (class design, severity BLOCKING, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D10: `lym machine start` starts the prime and owned beams of a hub this machine owns; under a prime another machine owns, or one with no recorded owner, it starts only the beams owned here, names the prime's machine, and claims no prime.
- D10: with a hold standing, `lym machine start` prints the hold's machine and subjects, and for this machine's own hold the `lym adopt <subjects>` re-run.
- D10: a hub's launcher is written at the first start on this machine of a dobby of a worktree this machine owns, a beam under a prime another machine owns included, stays while this machine owns anything in the hub, and is removed when it owns nothing there or the hub leaves `hubregistry.Live`; the all-hubs launcher goes with the last hub.
- D10: where no terminal launcher resolves, `lym machine start` prints each worktree's attach command.
