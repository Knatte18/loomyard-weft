---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: lens:affected-set
    location: _lym/plan/11-gate-adopt-key.md:5
    summary: seedAdoptGate inside MigrateConfig also runs on the local-layer gate file, sparseSeed and every run_settings or profile gate object, where PrepareRunSettings logs its added line at Warn on every read with a rename-or-remove way forward; Decision 9 licenses only a hub gate.yaml and the card settles none of these callers
    fix: "In card 11's Intent, name the other callers of gateslot.MigrateConfig (configsync.reconcileSparseFile's local-layer call at internal/configsync/configsync.go:424, sparseSeed at configsync.go:692, and configreg.PrepareRunSettings at internal/configreg/runsettings.go:73-79) and state for each whether it gains adopt. Recommended, since Decision 9 names only the hub gate.yaml, keep run_settings and profile documents unseeded and state how the card does it: either seedAdoptGate moves out of MigrateConfig into a step that only the hub base file's reconcile runs, with that call site added to the Edit list, or PrepareRunSettings skips the gate module's seed line. Then add a TestMigrateConfig or PrepareRunSettings row that pins the choice. If the author instead accepts seeding everywhere, the card says so and changes the Warn's way forward for an added key."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-dobby-worktree-gone.md:393
    summary: Every TestRun_WorktreeGoneExits row removes the worktree inside a tick, so the new exit test after the tick ends the loop first and no row reaches the top-of-iteration exit after a watch event; deleting that check would leave the suite green and the file-event acceptance criterion unpinned
    fix: In card 8's TestRun_WorktreeGoneExits, add a row that removes the worktree after tick 1 has ended, from inside the harness's Sleep seam, then wakes the loop by one watch event with no probe due. Assert that Run returns nil, that the fake records one StopStrand of the kept Tidy's guid, and that CheckBinary is not called again. Keep the existing in-tick rows.
  - id: F3
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/10-dobby-cli-exits-and-out.md:512
    summary: TestRunDobbyWith_TracesEachExitBeforeRun runs its rows in sequence over one hub, and its third row removes the prime's worktree link; the prime is the main worktree the task pairs hang from, so the later task-pair rows can no longer resolve or name the prime and their refused-as-neither-prime-nor-beam state cannot occur
    fix: In card 10's TestRunDobbyWith_TracesEachExitBeforeRun, run the row whose wire fake removes the prime's worktree link last, or on its own hubforge.CopyHub hub, and say so in the test description, so the task-pair and loose-directory rows run over an intact hub.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/07-dobby-start-and-exits.md:23
    summary: Card 7 lists internal/beamcli/doc.go under Extends while its change rewrites that file's existing dobby sentences, which say the verb registers before Run and refuses before anything is registered or locked
    fix: In card 7, move `internal/beamcli/doc.go` from the **Extends:** list to the **Edit:** list, so the Extends section is removed, and have the Docs line say that the dobby paragraph's registration order and the already_running sentence are rewritten. Checked with lym loom validate-plan --overlay, with no new finding.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/15-beam-status-adopt.md:746
    summary: ReadCheckView reads check.json through ReadCheck and state.ReadJSON, which creates the adopt scratch directory and a lock file, so every lym beam status call writes under _board/.lym/adopt and falsifies statusCmd's Long text that its one write is the empty orch directory
    fix: In card 15, have ReadCheckView stat check.json under fabricengine.AdoptScratchDir first and return nil without calling ReadCheck when it is absent, and have card 13's ReadCheck read through state.ReadJSONNoMkdir. If the author keeps the write, card 15's statusCmd Long text names it beside the orch directory.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/13-adopt-check-verb.md:645
    summary: A tier whose run refuses after Started returns through report's refusal branch before Ended is called, and card 13 clears Running only in Ended, so a check that ends failed can leave check.json and its lym beam status view showing a tier still in flight
    fix: In card 13's runCheck step 5, clear Running when the record is ended green or failed. Add to TestRunCheck a row whose fake gate calls Started and then returns 1 without Ended, asserting that the final record is failed with no Running.
  - id: F7
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: _lym/plan/08-dobby-worktree-gone.md:379
    summary: The kept-tidy refresh after each tick reads the record through ReadState, whose state.ReadJSON runs MkdirAll, so a removal between the marker test and that read recreates the .lym/beam stub at a site the decision's in-flight-tick bound does not cover
    fix: In card 8, read the record for the kept-tidy refresh with state.ReadJSONNoMkdir on Geometry.StatePath, which dobbyengine already imports, and treat a missing directory as the worktree gone. Alternatively, name this window beside the in-flight-tick bound in the doc.go bounds the card writes.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/07-dobby-start-and-exits.md:320
    summary: "The line dobby: stopped on error names no way forward, and on a fresh start, or when the re-exec state read failed, it is the only trace the dobby leaves, while every sibling notice names fix the cause, then run lym beam start"
    fix: "In card 7's Run steps 1 and 5, write the stop line as `dobby: stopped on error: <error>; way forward: fix the cause, then run lym beam start`, and have the TestRun_LockOwnerAndWakeups rows that assert the stop line assert that suffix."
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/10-dobby-cli-exits-and-out.md:512
    summary: Two TestRunDobbyWith_TracesEachExitBeforeRun rows re-cover inputs that TestDobbyCmd_RefusesAPathItCannotPlace already drives, a loose hub directory and a task pair without the beam mark; the new no-line and no-directory assertions belong in that test, and their no-notice half follows from TestReadsAsReexec's absent-directory row
    fix: "In card 10, drop the two rows from TestRunDobbyWith_TracesEachExitBeforeRun: the task pair without a .lym/beam directory, and the loose directory of the hub. Add `internal/beamcli_test#TestDobbyCmd_RefusesAPathItCannotPlace` to the **Edit:** list, and have each of its rows also assert that no dobby.log and no .lym/beam directory exist under the root. Checked with lym loom validate-plan --overlay, with no new finding."
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/11-gate-adopt-key.md:14
    summary: pattern/PATTERN-test-speed.md's Gate keys table lists each key with its site but gains no adopt row, and fabricengine.AdoptScratchDir's doc comment still says the directory holds only the suite log after card 13 adds check.json, check.out and check.lock there
    fix: "In card 11, add an **Extends:** list holding `pattern/PATTERN-test-speed.md`, and add to the Intent a Gate keys table row: adopt, lym adopt check, all. In card 13, add `internal/fabricengine#AdoptScratchDir` to the **Edit:** list, and have the Intent say its doc comment names the adopt check's record, output and lock beside the suite log. Both were checked together with lym loom validate-plan --overlay, with no new finding."
  - id: F11
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/02-batten-tells-machine.md:96
    summary: Card 2's refusal on an empty name, and the empty-name row of card 3's TestMachineLandings, guard a state the production seam cannot produce, since machinename.FromHostname already refuses an empty hostname
    fix: In card 2, drop the words "or returns an empty name" from the BuildShed refusal. In card 3, drop the empty-name row from TestMachineLandings and the empty-name half of machineLandings' refusal, leaving the failing-read row.
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/13-adopt-check-verb.md:645
    summary: Card 13 step 5 names busy slots as a way the gate ends before any tier, but RunGate sets --wait, under which acquireSlot blocks until the slots free and never answers slot busy
    fix: In card 13's runCheck step 5, change "as on a refused key or busy slots" to "as on a refused key".
---

### [MEDIUM] The adopt seed reaches every document MigrateConfig migrates

**Location:** `_lym/plan/11-gate-adopt-key.md:5`, `internal/gateslot#MigrateConfig` and `seedAdoptGate`.

**Issue:** Through `configreg`'s gate module row, `gateslot.MigrateConfig` runs on more than the hub base file:

- the full-file reconcile (`internal/configsync/configsync.go:507`);
- the sparse hub base file (`configsync.go:417`);
- the local-layer file (`configsync.go:424`, through `reconcileSparseFile` at 562-563);
- `sparseSeed` (`configsync.go:692`);
- `configreg.PrepareRunSettings` (`internal/configreg/runsettings.go:73-79`).

`PrepareRunSettings` logs every line a module's Migrate returns at Warn, and its doc (`runsettings.go:43`) says nothing rewrites a profile or a run_settings document, so the Warn repeats at every read.
A task or profile `gate` object that sets `gates.finalize` and no `adopt` would therefore warn `gates.adopt: added with the value of gates.finalize` on every read.
That warning's way forward reads "rename or remove the key in profiles.yaml or in the entry's run_settings", which is wrong for an addition.
The local-layer file would gain an `adopt` as well.
Decision 9 licenses the seed into "a hub `gate.yaml` that tunes `finalize`", and card 11 neither scopes the seed to that file nor states what the other callers do.

### [MEDIUM] No test reaches the worktree-gone exit at the top of the loop

**Location:** `_lym/plan/08-dobby-worktree-gone.md:393`, `TestRun_WorktreeGoneExits`.

**Issue:** Card 8 adds two exit tests:

- one at the top of each iteration, which Decision 1 asks for, so that a file-event wake exits;
- one after each tick, before `CheckBinary`.

Every planned row removes the worktree from inside the tick hook, which runs last in `tick` (`internal/dobbyengine/loop.go:304-309`), so the post-tick test always exits first.
No row reaches the top-of-iteration test, and the "one watch event, with no probe due" in the harness setup is never consumed.
The acceptance criterion "a beam dobby woken by a file event, not a probe tick, after its worktree is removed still exits" stays unpinned, and the suite stays green with that check deleted.

### [LOW] A test row breaks the hub the later rows run over

**Location:** `_lym/plan/10-dobby-cli-exits-and-out.md:512`, `TestRunDobbyWith_TracesEachExitBeforeRun`.

**Issue:** The rows run in sequence over one hub.
The third row's `wire` fake removes the prime's worktree link.
The prime is the hub's main worktree, and `fabricengine.PrimeName` (`internal/fabricengine/worktreelist.go:147`) lists worktrees through it.
After that row, the task-pair row "refused as neither prime nor beam" cannot reach that refusal.

### [LOW] Card 7 extends a doc it rewrites

**Location:** `_lym/plan/07-dobby-start-and-exits.md:23`.

**Issue:** `internal/beamcli/doc.go:97-100` says the dobby verb resolves the prime, registers through `registerDobby` and runs `dobbyengine.Run`, prints `already_running` when the lock is held, and refuses "before anything is registered or locked".
Card 7 moves the registration after the lock and drops the `Live` pre-check, so those sentences are rewritten, not added to.
`Extends` is for adding without changing what is there.

### [LOW] `lym beam status` gains writes its help says it does not make

**Location:** `_lym/plan/15-beam-status-adopt.md:746`.

**Issue:** `ReadCheck` (card 13) goes through `state.ReadJSON`, which runs `os.MkdirAll` on the record's directory and takes a read lock (`internal/state/state.go:96-101`).
`ReadCheckView` calls it on every `lym beam status`, so the verb creates `_board/.lym/adopt/` and a lock file even on a hub with no record.
That contradicts `statusCmd`'s Long text (`internal/beamcli/cli.go:72-73`), which card 15 edits only to name the new section.

### [LOW] A refused tier can leave `Running` set in a failed record

**Location:** `_lym/plan/13-adopt-check-verb.md:645`, with card 12's observer.

**Issue:** `execution.report` returns on `result.refusal` before writing a record (`internal/gatecli/run.go:448-451`), and card 12 calls `Ended` only past that branch.
Card 13 clears `Running` only in `Ended`, and step 5 ends the record without touching it.
A tier that refuses after `Started` thus leaves a failed `check.json` that names a tier in flight, and card 15 renders it.

### [LOW] The post-tick record read can recreate the removed worktree's stub

**Location:** `_lym/plan/08-dobby-worktree-gone.md:379`.

**Issue:** After the post-tick marker test passes, the loop reads the record through `ReadState` to refresh `keptTidy`.
`ReadState` goes through `state.ReadJSON`, which runs `os.MkdirAll` (`internal/state/state.go:96`).
A removal between the marker test and the read recreates `.lym/beam` at the removed path.
Decision 1's bound covers that recreation only for a tick in flight, not for this read the card adds.

### [LOW] The fatal-error stop line names no way forward

**Location:** `_lym/plan/07-dobby-start-and-exits.md:320`, steps 1 and 5 of `Run`.

**Issue:** A fresh-start fatal error, and a re-exec whose state read failed, post no notice.
In those cases `dobby: stopped on error: <error>` is the only trace in `dobby.log`, and it gives no next step.
Every sibling notice in the card names "fix the cause, then run lym beam start".

### [LOW] Two new test rows re-cover an existing test's inputs

**Location:** `_lym/plan/10-dobby-cli-exits-and-out.md:512`.

**Issue:** `TestDobbyCmd_RefusesAPathItCannotPlace` (`internal/beamcli/dobby_integration_test.go:21`) already drives a loose hub directory and a task pair without the beam mark through `lym beam dobby`, asserting the envelope, no run-process record and no lock.
Card 10's new test repeats both inputs only to add "no line, no notice and no directory".
The file assertions fit the existing rows.
The no-notice half needs no new row: with no resolved location nothing posts, and a missing `.lym/beam` directory reads as no re-exec, which card 7's `TestReadsAsReexec` pins.

### [NIT] Two docs still describe the gate keys and the adopt directory as they were

**Location:** `_lym/plan/11-gate-adopt-key.md:14` and `_lym/plan/13-adopt-check-verb.md:4`.

**Issue:** `pattern/PATTERN-test-speed.md`'s Gate keys table lists each key with its site, and no card adds `adopt`.
`fabricengine.AdoptScratchDir`'s doc comment (`internal/fabricengine/junctionnames.go:164`) says the directory holds the suite's log, and card 13 puts the check's record, output and lock there too.

### [NIT] The empty-name guards guard a state the seam cannot produce

**Location:** `_lym/plan/02-batten-tells-machine.md:96` and `_lym/plan/03-landings-by-machine.md:134`.

**Issue:** `machinename.Name` returns an error, never an empty name, because `FromHostname` refuses one (`internal/machinename/machinename.go:21-26`).
Card 2 says so itself, yet keeps an empty-name refusal, and card 3 adds an empty-name row and refusal half that only a fake reaches.

### [NIT] Busy slots cannot end a waiting gate

**Location:** `_lym/plan/13-adopt-check-verb.md:645`.

**Issue:** `RunGate` runs with `--wait`.
With `--wait` set, `acquireSlot` blocks without the slot-wait bound and never answers slot busy (`internal/gatecli/run.go`, `acquireSlot`).
Only a refused key ends the gate before any tier.

## Rejected

- Handler: card 2's `BuildShed` refusal reached only through the seam because the owner checks refuse an unknown machine first. Rejected: a continue pass returns from `battenPreRun` before the owner pre-flight (`internal/battencli/arm.go:731-735`), and dobby drives runs by continue passes, so the refusal is reachable in production.
- Lens guard-reach: card 6's Windows held-lock rotation branch, reachable only by a race. Not raised as a finding by the lens. Decision 5 licenses it, and `TestRotateOutBeforeSpawn` reaches it.
- Lens way-forward: card 8's `lym reed remove <guid>` line naming no directory to run it from. The lens left it out as a NIT, and the verb takes the guid positionally.
