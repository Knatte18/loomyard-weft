---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:guard-reach
    location: _lym/plan/21-dobby-catch-up.md (the hand_over trigger of daemon.drivePasses)
    summary: >-
      Card 21 owes the catch-up whenever drivePasses lists a run answering hand_over, but DobbyRuns lists every batten record
      on the prime whatever pair stands here, so a run another machine owns re-owes the catch-up on every tick without end.
    fix: >-
      In card 21's Intent, bound the fourth owing trigger to a listed run whose verdict answers hand_over and whose
      RunView.PairPresent is true, so a run already handed over (its local pair removed) or one that never had a pair here
      owes nothing. Add a TestRun_CatchUp row: a listed run answering hand_over with PairPresent false owes no catch-up and
      the next tick runs none.
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:affected-set
    location: _lym/plan/15-tender-bolt-hold.md (internal/tenderengine#Resume), with _lym/plan/13-act-rule-hold.md
    summary: >-
      Once card 13 fills Held, lym tender resume <slug> refuses on the act rule's held verdict before any lift, because
      tendercli's resumeAside sets no Hold, so card 15's bolt-hold resume rules and D5's lift-and-replace rule are unreachable from the verb.
    fix: >-
      Card 15 adds `internal/tendercli#resumeAside` under Edit and sets Hold in it (runfsm.Aside{Pause, UnfinishedCancel,
      ShedPause, PushOwed, Hold}), so the resume's verdict sets the hold aside and Resume's own bolt-hold rules decide.
      State it in the Intent beside the HubDeps wiring, and add a TestResume row whose Verdict seam answers held under a
      standing hold, asserting the slug's own entry is lifted and a bolt-hold entry written in its place.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/08-dobby-start-owner-flags.md and _lym/plan/23-retire-owner-pause-callers.md (Edit lists)
    summary: >-
      Cards 8 and 23 reshape or remove dobbyengine, battenshed and loomcli seam fields, yet leave unlisted the test
      helpers and tests that still set the old shapes, so neither card's packages compile at its own gate.
    fix: >-
      Card 8 adds under Edit `internal/dobbyengine#runAnAdoptHold`,
      `internal/dobbyengine#TestRun_ALaterArchiveFailureLogsAgainAfterTheOwnerLeaves`,
      `internal/dobbyengine#TestRun_AFailedBoltPullLeavesTheOwnerReadToTheLocalBolt`,
      `internal/dobbyengine#TestRun_BoltPullBringsInANewOwner`, `internal/dobbyengine#TestRun_LockOwnerAndWakeups`,
      `internal/dobbyengine#TestRun_TickFailuresLogOncePerEpisode` and `internal/dobbyengine#fakeStart.baseDeps`, each
      switching its Owner or PrimeOwner fake to the (owner, elsewhere, err) shape. Card 23 adds under Edit
      `internal/dobbyengine#newPassHarness`, `internal/dobbyengine#runAnAdoptHold`, `internal/battenshed#newLockHarness` and
      `internal/loomcli#stoppingPrePushCheck`, each dropping the PauseOwned, LiftOwnerPause, Machine or pauseOwned field it sets.
      The method behind it: for every seam field a card reshapes or removes, list every test member that assigns it, not only the harness.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/16-held-tasks-not-started.md (internal/battencli#battenCLI.refuseStartOwnedElsewhere)
    summary: >-
      Card 16's held refusal in refuseStartOwnedElsewhere is shadowed by the upstream check refuseByAct
      (internal/battencli/arm.go:662), which startFirstPass runs first and which card 13's Held already makes refuse,
      so only the direct test call reaches it.
    fix: >-
      Drop `internal/battencli#battenCLI.refuseStartOwnedElsewhere` and `internal/battencli#TestRefuseStartOwnedElsewhere`
      from card 16's Edit list, and delete the batten-start paragraph and the TestRefuseStartOwnedElsewhere test line from
      its Intent. In their place, state that `lym batten run <slug>` under a hold is refused by refuseByAct with card 13's
      held refusal, which TestAct and TestReadRun cover, and that the fix task's start passes because ReadRun reads it not held.
  - id: F5
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/08-dobby-start-owner-flags.md (internal/hubgeom#DobbyStart, internal/dobbyengine#EnsureLive)
    summary: >-
      D3 says a registration refused at a dobby start posts one notice and shows in lym beam status, but card 8 only
      lets EnsureLive's name error carry an unresolved config directory's refusal, which posts nothing and leaves no
      status; card 21's refused notice for that case is reached only by an already-running dobby.
    fix: >-
      In card 8's Intent, make EnsureLive report the identity refusal as an OutcomeRefused entry for each named worktree.
      Its Detail names the variable to set and the lym beam stop and lym beam start restart.
      State that DobbyStart's caller in internal/beamcli posts that refusal once to the nearest live orch,
      and that lym beam status shows the refused start entry. Add a TestEnsureLive row whose Machine seam returns the
      refusal and that asserts the refused entries and their detail.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/09-this-machine-names-registry.md (plan:internal/pairteardown#machineNamesOf)
    summary: >-
      Card 9 gives a machine with no registry entry its file's names alone, so its takeover read and the landing's
      ThisMachine().Holds comparison read a registered machine's current or former name as its own, against D3 and card 3's ReadsAsHere.
    fix: >-
      In card 9's Intent, make machineNamesOf leave out of Current's companions every name a registry entry of another id
      holds as its current or former name when the registry holds no entry under the identity's id, matching
      Registry.ReadsAsHere; and add a TestMachineNamesOf row where an unregistered identity's name is another entry's
      current name and the returned set does not hold it.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: handler
    location: _lym/plan/21-dobby-catch-up.md (daemon.syncBoltHold, internal/beamcli#hubDobbyDeps)
    summary: >-
      No card carries D5's rule that a run holding its parent lock gets no bolt-hold slug entry; card 21's held list is
      every listed run whose verdict reason is held, so such a run's step in flight would be held by the gate.
    fix: >-
      In card 21's Intent, make hubDobbyDeps leave out of the held list each run whose parent lock record
      (internal/parentlock, under the board's parent lock directory) names the run as its holder,
      so that run gets no bolt-hold entry while the act rule still starts no producer for it.
      Add a TestRun_SyncsBoltHoldEntries row for a held run holding its parent lock, left out of the held list.
  - id: F8
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/plan/19-pause-intent-entries.md (internal/tendercli#commitRunIntent)
    summary: >-
      Card 19's Intent gives commitRunIntent a new flag parameter, but its Edit bullet carries no re-sign arrow, and
      hubRunIntents, whose closures call it and must pass the flag, sits under Uses instead of Edit.
    fix: >-
      Replace card 19's Edit bullet with `internal/tendercli#commitRunIntent` -> `func commitRunIntent(prime
      *lymcwd.Location, slug, message string, stderr io.Writer, returnOwed bool, write func() (found bool, err error))
      error`, add `internal/tendercli#hubRunIntents` under Edit, and drop it from Uses.
  - id: F9
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/17-hand-over-owned-elsewhere.md (Intent)
    summary: >-
      Card 17 adds a routing edge that force-removes a local pair on a registry owner reading with no takeover entry on
      origin, but its Intent never states what this lets through or what bounds it.
    fix: >-
      Add a Bound paragraph to card 17's Intent. It hands over only a run whose owner reads elsewhere after the
      catch-up's registration landed (card 18 step 1), so a refused or unregistered identity never reaches it.
      The local pair's forced removal discards uncommitted files only after the archive tags naming its tips are on origin.
      It never moves an origin branch, writes the board or touches another machine's records.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/23-retire-owner-pause-callers.md and _lym/plan/25-pattern-owner-machine.md
    summary: >-
      Retiring the owner pause in card 23 leaves text describing it in pauserecord/doc.go, machinecli's user-facing
      recoveries, PATTERN-dobby-runs.md, PATTERN-owner-machine.md and adoptcli/doc.go, and no card lists those texts.
    fix: >-
      Card 23 adds under Edit `internal/pauserecord/doc.go` (drop the owner-pause lifter and the dobby's owner-pause lift),
      `internal/machinecli#recoveries` (drop the line saying the dobby's probe lifts an owner pause) and
      `pattern/PATTERN-dobby-runs.md` (drop the dobby's owner lift from the Adopt pause rule), and adds
      `internal/machinecli/doc.go` under Uses. Card 25's Intent names the PATTERN-owner-machine.md lines it rewrites:
      An entry owned elsewhere is paused under the owner, and the Enforcement clause on InnerRun's owner pause.
      The stale adoptcli/doc.go lines (33 and 210) get no proposed target:
      listing that file under card 23 trips uses-later-target, since cards 1, 6 and 7 list it under Uses.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/11-forget-former.md (plan:internal/machinecli#hubReferences)
    summary: >-
      Card 11's hubReferences calls pairteardown.ReadBattenRecord, which falsifies PATTERN-cli-cobra.md's line that
      machinecli calls only machinename, boardengine, fabricengine and hubregistry.
    fix: >-
      Card 11 adds `pattern/PATTERN-cli-cobra.md` under Edit and adds pairteardown to that background's machinecli line.
  - id: F12
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/00-overview.md (Decision absences and their observers), _lym/plan/20-hand-pass-catch-up.md
    summary: >-
      The shared decision rests on the premise that batten cannot tell a hand pass from a dobby's full pass, yet the
      dobby spawns its passes with flags it chooses; and running the catch-up on every full pass lets a pass's catch-up
      race the dobby's own over one pair, which the same decision's rationale says it avoids.
    fix: >-
      Rewrite the shared decision's rationale to drop the false premise and say the plan runs the catch-up on every
      non-continue pass by choice. Extend card 20's Bound to say why a pass catch-up racing the dobby's catch-up over one pair is safe.
      Pushes go under the push lock, a hand-over stops at this machine's hand-over mark, and pair removal takes the teardown lock.
---

### [BLOCKING] The hand_over trigger re-owes the catch-up on every tick

**Location:** `_lym/plan/21-dobby-catch-up.md`, the fourth owing trigger ("when `drivePasses` lists a run whose verdict answers `hand_over`").

**Issue:** `battencli.DobbyRuns` (`internal/battencli/dobbyreads.go:158`) lists every batten record under the prime whose parent is the dobby's worktree.
It never filters on whether the pair stands here: `PairPresent` is only read into the view.
A run another machine owns answers `hand_over` on every read (`internal/runfsm/act.go:111`), and that includes:

- a run this machine already handed over and whose local pair the hand-over removed;
- a run whose pair never stood here.

Under card 21 each such tick owes the catch-up again.
The next tick runs `CatchUp` again: a Bolt pull, a pull-first registration and the pair fetches.
While the catch-up is owed, the tick spawns no pass.
The loop never settles while such a run is listed, so other machines' runs in the same parent worktree can stall this dobby's scheduling and load origin every tick.

### [BLOCKING] Tender resume refuses on the act rule's held verdict before card 15's hold rules

**Location:** `_lym/plan/15-tender-bolt-hold.md` (`internal/tenderengine#Resume`), with `_lym/plan/13-act-rule-hold.md`.

**Issue:** `tenderengine.Resume` checks `req.Verdict()` and returns its refusal before any lift (`internal/tenderengine/resume.go:122-128`).
The verb fills that verdict from `hubRunIntents`, judged with `resumeAside` (`internal/tendercli/runintents.go:31`), which sets no `Hold`.
Card 13 makes `Held` true for every slug while a hold stands, so `lym tender resume <slug>` under a hold always returns runfsm's generic held refusal.
Card 15's rules are then reached only by `TestResume`'s fakes:

- the refusal of a slug that only a `bolt-hold` entry covers;
- the lift of a slug's own entry that writes a `bolt-hold` entry in its place.

D5's rule "lifting a slug's own entry under a standing hold writes the `bolt-hold` entry in its place" is not delivered.

### [BLOCKING] Cards 8 and 23 leave tests that set the old seam shapes unlisted

**Location:** `_lym/plan/08-dobby-start-owner-flags.md`, `_lym/plan/23-retire-owner-pause-callers.md`.

**Issue:** Card 8 changes `Deps.Owner` and `StartDeps.PrimeOwner` to return an elsewhere flag.
These tests and helpers outside the card's list still assign the old shape:

- `internal/dobbyengine/handover_test.go:163`, `:289`, `:321` and `:352`;
- `internal/dobbyengine/loop_test.go:210`, `:229`, `:242` and `:697`;
- `fakeStart.baseDeps` (`internal/dobbyengine/start_test.go:65`).

Card 23 removes these seams, which tests outside its list still set:

- `Deps.PauseOwned`, `Deps.LiftOwnerPause` and `Deps.Machine`: `newPassHarness` (`passes_test.go:223/229/239`) and `runAnAdoptHold` (`handover_test.go:178`);
- `InnerRunDeps.PauseOwned`: `newLockHarness` (`internal/battenshed/innerrun_parentlock_test.go:97`);
- `prePushCheck.pauseOwned`: `stoppingPrePushCheck` (`internal/loomcli/prepush_test.go:392`).

Field changes are not re-signed members, so the gate's `caller-uncovered` check does not catch them.
Each card's test build breaks at its own gate.
The proposed targets passed `lym loom validate-plan --overlay` together with every other target fix in this review.

### [MEDIUM] Card 16's batten held refusal is shadowed by refuseByAct

**Location:** `_lym/plan/16-held-tasks-not-started.md`.

**Issue:** `startFirstPass` runs `refuseByAct` before `preflightRunOwner` (`internal/battencli/cli.go:567-575`).
`refuseByAct` (`internal/battencli/arm.go:662`) judges the act rule over `ReadRun` with `runAside`, which sets no `Hold` (`arm.go:638`).
Once card 13 fills `Held`, a start under a hold is refused there with runfsm's held refusal, which names the hold's machine.
So the new clause in `refuseStartOwnedElsewhere` is reached only by `TestRefuseStartOwnedElsewhere`'s direct call.
The refusal is a duplicate, and its test pins a path no entry point takes.

### [MEDIUM] The registration refusal at a dobby start posts no notice and shows no status

**Location:** `_lym/plan/08-dobby-start-owner-flags.md`.

**Issue:** D3 says "a registration refused at a catch-up or dobby start stops it: the machine acts on no run, posts one notice naming the refusal and its way forward, and `lym beam status` shows it".
Card 8 routes an unresolved config directory through `StartDeps.Machine`, which returns `EnsureIdentity`'s name.
`EnsureLive` then returns its "this machine's name is unknown" error (`internal/dobbyengine/start.go:127-130`).
That reaches only `lym beam start`'s caller: no notice is posted, and with no dobby running, `lym beam status` shows nothing of the refusal.
Card 21's `refused` notice, which names the variable, is then reached only by a dobby already running from before the upgrade.

### [MEDIUM] machineNamesOf reads a registered machine's name as this unregistered machine's own

**Location:** `_lym/plan/09-this-machine-names-registry.md`.

**Issue:** D3 says "a machine with no registry entry reads no registered machine's current or former name as its own".
Card 3's `ReadsAsHere` follows that rule.
Card 9 says "A machine with no entry gets its file's names alone".
The set it builds feeds two comparisons:

- `fabricengine`'s takeover read (`MachineNames.Holds`, `internal/fabricengine/takeover.go:117`);
- loom's landing owner check, `owner != "" && !here.Holds(owner)` (`internal/loomcli/landingdeps.go:162-173`).

So an unregistered machine whose name is a registered machine's current name reads that machine's takeover entry and ownership as its own.
These comparisons go around the registry, against D3's "every comparison ... goes through the registry".

### [MEDIUM] No card carries the parent-lock exemption from bolt-hold entries

**Location:** `_lym/plan/21-dobby-catch-up.md`.

**Issue:** D5's Bound says "A run holding its parent lock gets no slug entry, so the gate does not hold its step in flight; the act rule still starts no producer for it."
No card mentions it: a search of the cards for "parent lock" finds only card 23's owned-elsewhere release.
Card 21 hands `SyncBoltHold` every listed run whose verdict reason is `held`, and card 15 writes an entry for each.
A run in the middle of its parent-locked step therefore gets a `bolt-hold` entry, and the gate holds that step while it holds the parent lock.

### [MEDIUM] Card 19 re-signs commitRunIntent without the arrow and leaves its caller under Uses

**Location:** `_lym/plan/19-pause-intent-entries.md`.

**Issue:** The Intent says "`commitRunIntent` gains a flag saying whether an owed push is answered on stderr (the verbs) or returned (this caller)", a parameter change.
The Edit bullet `internal/tendercli#commitRunIntent` carries no re-sign arrow.
Its call sites are the closures in `hubRunIntents` (`internal/tendercli/runintents.go:63`, `:76`).
`hubRunIntents` is listed under `Uses`, so the card claims to leave it unchanged while it must pass the new flag.
The proposed arrow and target move passed the overlay check.

### [MEDIUM] Card 17 states no bound for its forced local pair removal

**Location:** `_lym/plan/17-hand-over-owned-elsewhere.md`.

**Issue:** Until now `HandOver` ran only when origin held another machine's takeover entry for the run (`internal/pairteardown/handover.go:57`).
Card 17 adds a new route into it, driven by a board owner reading through the registry alone.
That route ends in `RemovePair` with `Force: true` (`internal/pairteardown/handover.go:118`).
The Intent says what the removal skips (no origin step, no board write), but not what bounds the route:

- that a refused or unregistered identity cannot reach it;
- that uncommitted files are discarded only once the archive tags have landed.

The rubric's attack-surface question requires that bound in the Intent.

### [MEDIUM] Text describing the retired owner pause stays after card 23

**Location:** `_lym/plan/23-retire-owner-pause-callers.md`, `_lym/plan/25-pattern-owner-machine.md`.

**Issue:** After card 23 no code writes or lifts an owner pause, but these texts still describe it, and no card edits them:

- `internal/pauserecord/doc.go:17-28`: "its lifter", "A dobby's owner-pause lift and the hibernate wake judge only `OwnerCaller`". Card 12 edits this doc earlier, while the text is still true.
- `internal/machinecli/set.go:35`: the user-facing recovery "the dobby's probe lifts that owner pause". This is printed operator guidance.
- `pattern/PATTERN-dobby-runs.md:21`: "the dobby's owner lift".
- `pattern/PATTERN-owner-machine.md:21`: "An entry owned elsewhere is paused under the owner". Line 78 says the tests "cover `InnerRun`'s owner pause". Card 25 says only that "its sections change with the code".
- `internal/adoptcli/doc.go:33` and `:210`: these name the owner pause. A target for this file under card 23 fails `uses-later-target`, because cards 1, 6 and 7 list it under `Uses`, so this review proposes none.

### [LOW] Card 11 makes PATTERN-cli-cobra's machinecli line false

**Location:** `_lym/plan/11-forget-former.md`.

**Issue:** `pattern/PATTERN-cli-cobra.md:64` says `machinecli` "calls the functions of `machinename`, `boardengine`, `fabricengine` and `hubregistry`".
Card 11's `hubReferences` reads `pairteardown.ReadBattenRecord`, and no card updates the line.

### [LOW] The absences decision rests on a false premise

**Location:** `_lym/plan/00-overview.md`, Decision "absences and their observers", and `_lym/plan/20-hand-pass-catch-up.md`.

**Issue:** The rationale says "batten cannot tell a hand pass from a dobby's full pass".
Yet the dobby spawns every pass itself with flags it chooses (`internal/beamcli/dobbydeps.go:563-573`: `--continue-pass`, `--no-reopen`).
The same rationale says "one observer per worktree avoids two catch-ups racing over one pair".
Card 20 nonetheless runs a catch-up in every dobby full pass, and that can run beside the dobby's own catch-up when a sleep is seen while a pass is in flight.
D4 asks for the hand pass's catch-up only, so the wider choice and its race need stating, not a false premise.

## Rejected

- guard-reach finding 4 (card 21's refused notice reachable only on a narrow path) is folded into F5, which raises the D3 gap behind it.
- affected-set F2 and repo-invariants F1 describe the same unlisted-test defect and are merged into F3; repo-invariants F2, F3, F4 and F6 are merged into F10 as one finding about the method.

## Focus departures

- "Granularity and test economy: cards 7, 20, 21 and 23 are the largest; check for per-symbol tests and for tests duplicating coverage that already exists, such as TestReadRun_ReadsTheSameStateWithoutTheCache." I checked them and raise no finding.
  Card 14 reuses that test's fixture shape for a new behaviour, the pull before the read, and adds no duplicate.
  Card 8's TestBeamPair pins the D3 criterion that names OwnedWorktrees, so it is not redundant with TestReadsAsHere.
- "Attack surface: the new --forget-former verb (card 11), the hold write (card 5) and the local-mode pair removal in the hand-over (card 17)". Card 11 states its bound, and card 5's Intent states that only the upgrade's first read calls it and that another machine's standing hold wins.
  Only card 17 lacks a bound, which F9 raises.
