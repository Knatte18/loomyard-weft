## Goal

A machine registry and holds in Bolt, the owner pause caller retired, and a catch-up from origin after an absence

Takes: runfsm's Act rule, the synchronous Bolt writes, the takeover reader and hand-over and the owed-push facts from push-before-act-takeover; the subject-process end from orch-resume-own-session; boardengine's owner setter and OwnedElsewhere, machinename's former names, pauserecord's callers and SleepWatch, the dobby's loop and handover, battencli's origin-owed marker, pairteardown's local mode
Delivers: board.json's machines registry and holds list written only by boardengine's setter with a synchronous push; every machine-name comparison through the registry, OwnedElsewhere's callers in every package updated; lym machine name --forget-former; hold slug entries under the new pauserecord caller hold, read by every machine; the owner caller and its writers retired into hand_over with the subject-process end; the origin-owed marker retired; the catch-up from origin after a sleep, a dobby start or a hand pass, pushing owed commits, handing over runs owned elsewhere and removing their local pairs in local mode, the dobby's handover calling the subject-process end
Exports: boardengine's registry and hold types, setters and readers, OwnedElsewhere taking the registry, and the catch-up entry point adopt calls as its step 4

## Scope

The part changes these packages:

- internal/boardengine (about 600 changed lines)
- internal/dobbyengine (about 450 changed lines)
- internal/pauserecord (about 120 changed lines)
- internal/machinecli (about 150 changed lines)
- internal/orchcli (about 120 changed lines)
- internal/tenderengine (about 100 changed lines)
- internal/battencli (about 140 changed lines)
- internal/beamcli (about 100 changed lines)
- internal/tendercli (about 60 changed lines)
- internal/hibernateengine (about 60 changed lines)
- internal/adoptcli (about 40 changed lines)
- internal/machinename (about 40 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/battenshed (about 30 changed lines)
- internal/landingshed (about 10 changed lines)
- internal/fabricengine (about 10 changed lines)
- internal/shedrecipe (about 10 changed lines)
- internal/shedrun (about 10 changed lines)
- internal/hibernatecli (about 10 changed lines)
- internal/fabriccli (about 10 changed lines)
- internal/fronttask (about 10 changed lines)
- internal/pairteardown (about 10 changed lines)
- internal/pawlengine (about 10 changed lines)
- pattern (about 10 changed lines)

## Decisions

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
   pushes the code tip the run's last transition on origin names when origin lacks it (D4),
   and archives the local tips as D4 does, the local parent branch's unpushed merge of this run included.
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
  A hold has a kind, `adopt` or `adopt-fix`, the machine, the time, its adopt's subjects,
  and for `adopt-fix` the fix task's slug and each subject's try count (D8).
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
  When at least one entry became a pause intent or belonged to a released `queued` task, that first read also writes one Bolt `adopt` hold
  naming this machine and the subjects of those runs and tasks, so no queued task starts;
  a re-run of `lym adopt` of those subjects here clears it at its green check.
  That first read posts one notice to the operator naming the hold, its machine,
  and as the way forward a re-run of `lym adopt` on that machine of the hold's subjects.
  A first read whose entries all belong to landed runs or runs owned elsewhere writes no hold, so every upgrade hold names subjects to re-run.
  A first read that finds another machine's hold standing writes none, so one machine holds the hub at a time;
  its notice names that hold's machine, and the re-run here once that hold clears.
  Like every notice an agent reads, neither names `--take-hold`.
  A released task's entry with no batten record is dropped, and the task stays `queued` under the hold.
- The readers of the retired adopt record that ask whether an adopt here covers a worktree, the dobby's handover and the orch watcher among them,
  read a hold this machine wrote.

Bound: a machine other than the adopter learns of a hold at its next Bolt pull or catch-up:
its dobby's pull within the dobby's pull interval, and a verb's before it asks the act rule.
Until then its dobby goes on with its runs as before, for at most the dobby's Bolt pull interval; D4's bound covers the transitions it pushes.
A verb that cannot reach origin reads the local Bolt, and any transition it leads to waits on origin under D2.
A run that holds its parent lock is exempt from slug entries, so the gate does not hold its step in flight;
the act rule still starts no producer for it.
A hold stands until a green check clears it: that of the adopt holding it, or the fix task's re-check under `adopt-fix` (D8);
a hold whose adopter stopped stands until the operator takes it over with `lym adopt --take-hold` on another machine (D8).

Rationale: the operator decided nothing happens in the hub until adopt is done and its failing tests are fixed;
the pause record is machine-local, so no other machine sees a hold kept there (state map 3.2, 6.4 items 7 and 11).

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

### Added after Discussion (arbiter, 2026-11-15): Round 3 review findings answered by the round 3 fixes

Decision: The Discussion review ends at its hard cap with round 3's findings answered by the record as the round 3 fixer left it, and no further edit is owed for them. F1: D1 reads past the parent-side merge from origin's fetched parent branch, D2 and D4 push the parent branch before the Finalize's transition and its board done, and D4 covers a takeover between the local merge and the parent push. F2: --release is gone; D8's lym adopt --take-hold, D7's hold subjects, the shell-guard row and their acceptance lines replace it. F3: D10 keeps only the same-machine resume. F4: Open risks reads None. F5: D2 puts records-first order in fabricengine.PushPairAnchored for every caller, and D4 hands over on any caller's rejected records push. F6: D3 deletes a cancelled run's batten record in the same step as every pair removal, and D1 reads a leftover cancelled record whose task branch is gone from origin as no record. F7: D2 turns a teardown-lock wait past StatusPushLockWait into an owed push, and D7 writes the upgrade hold only when a subject remains to re-run.

Rationale: Each fix in the record is the one the round 3 review proposed, and the code it names holds: finalize.go marks the board done (line 445) before pushParent (line 786), replay.go's finalizeEntered and ParentHoldsTaskWork read local state, PushPairAnchored iterates a code and a records side, battencli removePairAndBranch discards the task branch on origin, pauserecord.Admit, fabricengine.StatusPushLockWait and pairteardown Request.Local exist. The fixer disputed and deferred nothing.

## Constraints

- `PATTERN-owner-machine`: the owner stays on roots and beams, written only by `boardengine`'s setter, read through `hubgeom.ParentOf`.
  The rest of a Finalize past its parent-side merge, as origin's parent branch shows it,
  finishes on the run's owner, the adopter after a takeover (D1, D4).
- `PATTERN-pair-remote`: the code branch still goes along with a pair's records push, now once the records side lands (D2),
  and a refused push still leaves a failed-push record and a notice.
  A catch-up's local pair removal (D6) goes through `internal/pairteardown`'s local mode and deletes no origin branch.
- `PATTERN-dobby-runs`: runs are still started only by the parent's dobby, which starts only with its orch bar `--no-orch`.
- `PATTERN-lym-files`: every new tracked file sits under `_lym` and every cache under `.lym`, both named in `internal/lymdirs`.
- `PATTERN-tolerant-records`: the new tracked records and the Bolt `machines` and `holds` keys read tolerantly and are written through `internal/state` or the board store.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: the synchronous push, the takeover entry's push and the archive run in `internal/fabricengine` through `gitexec` and `gitrepo`.
- `PATTERN-shuttle-provider-seam`: Claude's transcript paths and the resume check stay in `internal/shuttleengine/claudeengine`.
- `PATTERN-child-processes`: the orch's Claude process registers in `internal/runproc` (D9), and D9's ends go through it.
- `PATTERN-daemon-wakeups`: D2's and D6's retry waits wake at most once a second and back off while idle.
- `PATTERN-refusal-way-forward`: every refusal the act rule, the holds, D3's owner refusal, D5's clash refusal and D9 add
  names its way forward and a test reaches it;
  none an agent reads names `lym adopt --take-hold`, which a shell-guard row denies agents (D8).
- `PATTERN-shell-guard-table`: the deny of `lym adopt --take-hold` is a row of `internal/shellguard`'s table.
- `PATTERN-code-conventions`: the orphan grace and the code-tip grace are each one named constant.

## Auto-mode assumptions

- The code-tip grace is 30 minutes (D4) and the orphan grace 24 hours (D11).

## Open risks

## Acceptance criteria

- D2: a run claim, a landing's board `done`, an owner write and a hold write are on origin when their verb returns.
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
- D7: `lym loom resume` on a worktree whose dobby is down refuses a run under a hold another machine wrote since that worktree's last Bolt pull.
- D9: a subject read as owned elsewhere leaves no process of its runs, its orch or its dobby running on this machine.

## PATTERN changes

- `PATTERN-owner-machine` — changes: Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first.
