---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/plan/02-runproc-owner-record.md:60
    summary: End and EndProcess check a start time only before the first signal of a record, never for a read descendant nor before SIGKILL, so a pid reused during the grace can be killed, against D3's bound
    fix: In card 2's End paragraph, state that every signal to a read descendant (pid or group) first compares the start time read with it, that the grace poll counts a pid whose start time differs from the read one as gone, and that the SIGKILL phase repeats each start-time and group check before signalling; state the same for EndProcess's SIGKILL. Add TestEnd rows for a descendant whose pid reads a different start time at the SIGTERM and at the SIGKILL, and an endProcess row for a pid reused between SIGTERM and SIGKILL, each asserting no signal is sent to it.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: lens:guard-reach
    location: _lyx/plan/11-fabric-remove-reconcile.md:32
    summary: A pair whose task worktree is gone skips the reconcile and runs today's non-local removal, so an ended or foreign run still gets the archive-tag push and remote step D5 and #741 withhold
    fix: In card 11, make the gone-worktree path take localReading (the local Bolt's owner and done reading the card already builds for an unreachable origin) in place of the reconcile, so owned_elsewhere and ended still remove through the local request with the dropped --remote named; add a TestRemover row for a gone task worktree with a local foreign owner that runs the local request, and keep the row with neither reading running today's request.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/17-shuttle-shell-only-waiting.md:29
    summary: The gated arrival returns OutcomeDone whatever the output files, while a plain gated turn end with an output missing is held, so it departs from D11's evaluate the gate as at a plain turn end
    fix: In card 17's Gated run paragraph, return OutcomeDone for a gated waiting turn end with only unawaited shells only when every output file exists, and otherwise return the held turn end of that waiting turn end (its message and offset) as pollEventsTick does at a plain turn end; add a TestPollEventsTick_OutstandingBackgroundWork row for a gated run with an output missing and a non-awaited shell that expects the held turn end.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lyx/plan/10-teardown-skip-readings.md:53
    summary: A waiting marker of any registered pid of the slug lets the run-lock holder pass even when the holder is a known step merging under the parent lock, so End kills a mid-merge step that D5 says refuses
    fix: In card 10, let a marker naming a registered pid other than the holder pass only when the holder is unknown (no unfinished step record) and no parent lock record names the slug; a marker naming the holder's own pid passes as written. Add TestRunLockHolderPasses rows for a known step holder with a marker of another registered pid and a parent lock record (refuses) and the same with no parent lock record but a known holder (refuses).
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/10-teardown-skip-readings.md:54
    summary: A paused holder refused because a parent lock record names the slug is told to retry once its pid ends, but under the pause the step never ends and batten keeps the record, so the retry never comes due
    fix: In card 10, word the paused-with-parent-lock refusal's way forward as resuming the run (lyx tender resume <slug>) so the step finishes its merge, then the removal again; card 12's wrap names lyx reed down in place of the removal. Assert the wording in the TestRunLockHolderPasses or TestTeardown_CallSequence refusal row.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/10-teardown-skip-readings.md:54
    summary: The unknown-holder refusal says retry once lyx tender status lists no owned process of the slug, but a paused run's parked driver, loop and batten stay listed until this very removal ends them, and an unknown holder is unregistered
    fix: In card 10, word the unknown-holder way forward as card 9's driver-lock refusal does, naming the run lock path and finding its holder in lyx tender status's replaced-binary part or with ps, ending it, then the removal again (lyx reed down again in card 12's wrap); update the refusal row's assertion.
  - id: F7
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/17-shuttle-shell-only-waiting.md:46
    summary: internal/websterengine/doc.go still says no background shell expires and a gated turn end waiting on a shell is never an arrival, which card 17 makes false, and no card edits it
    fix: Add `internal/websterengine/doc.go` to card 17's Edit list (checked with validate-plan --overlay) and state in its Docs paragraph that the Background shells in Master's wait section is reworded to D11, the awaited prefix keeping a shell holding the gated turn end and background_shell_wait_min expiring an ungated wait.
  - id: F8
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/02-runproc-owner-record.md:50
    summary: Register writes through state.WriteJSON, which takes a lock path the card never names, so each record leaves a lock file the prune never removes and List must tell apart from records
    fix: In card 2's Record paragraph, name Register's lock path (the record path plus .lock), state that List reads only .json names and removes a pruned record's lock file with it, and add a TestRegisterAndList assertion that a pruned record leaves no lock file.
  - id: F9
    severity: LOW
    class: design
    origin: lens:way-forward
    location: _lyx/plan/11-fabric-remove-reconcile.md:35
    summary: The stopped refusal appends then lyx fabric remove again to the entry's way forward, but for a live unpaused pair that way forward names no step now, so the retry cannot succeed
    fix: In card 11, when the stopped entry's branch reads live, name lyx tender pause <slug> before lyx fabric remove <slug> again in the refusal, and add the row to TestRemoveVerdict.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/13-owned-elsewhere-end.md:57
    summary: An End error or survivors at the owned-elsewhere sites are logged at Warn with no next step named
    fix: In card 13, have PauseOwnedElsewhere's and holdRuns' Warn lines name the way forward, lyx fabric remove <slug> or lyx reed down in the pair while it exists and lyx tender reap once it is gone.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/12-reed-down-ends-owned.md:43
    summary: downCmd's Long help still says down kills this worktree's session unconditionally, with no word of the refusal of a running unpaused run or ended_processes
    fix: In card 12, state that downCmd's Long text gains the pair readings, the refusal naming lyx tender pause <slug>, and the ended_processes key.
---

### [MEDIUM] End and EndProcess can signal a reused pid

**Location:** `_lyx/plan/02-runproc-owner-record.md:60`
**Issue:** The pass checks start times for leaders and non-leaders before SIGTERM, but a read descendant is signalled with no start-time check, the grace poll counts any live pid as not gone, and "SIGKILL what is left the same way" never re-reads identity. A descendant that exits during the five-second grace and whose pid is reused would be killed. D3's bound is that every descendant is "matched by start time". `EndProcess` has the same gap between its SIGTERM and SIGKILL. No TestEnd row pins either case.

### [MEDIUM] fabric remove of a gone task worktree skips the owner guard

**Location:** `_lyx/plan/11-fabric-remove-reconcile.md:32`
**Issue:** The run-scope reconcile refuses a slug with no pair worktree (`internal/fabricengine/originreconcile.go:574-577`), so the card skips it and runs today's non-local `Remove`, which pushes the archive tag and runs the remote step (`internal/pairteardown/teardown.go:137-145`). D5 removes an ended or owned-elsewhere run in local mode so that a non-owner pushes no stale records (#741). The card's `localReading` already gives that verdict from the local Bolt.

### [MEDIUM] Gated arrival ignores missing output files

**Location:** `_lyx/plan/17-shuttle-shell-only-waiting.md:29`
**Issue:** At a plain turn end, `pollEventsTick` returns Done only when every output file exists and otherwise holds the turn end (`internal/shuttleengine/wait.go:849-858`). Card 17's gated branch returns Done whatever the files, so a writer that stopped with an output missing and a stray shell goes to gate evaluation instead of being held. D11 says "the gate is evaluated as at a plain turn end".

### [MEDIUM] Marker pass lets End kill a step merging under the parent lock

**Location:** `_lyx/plan/10-teardown-skip-readings.md:53`
**Issue:** A Finalize's parent-locked push is registered under the slug (cards 3 and 7) and waits at the gate under a hub entry, writing a waiting marker (card 5). The run-lock holder then passes on that marker even when the holder is a known step still merging under the parent lock. The paused case excludes a parent lock record, but the marker case does not. D5 says such a holder refuses.

### [MEDIUM] Paused parent-lock holder's way forward cannot progress

**Location:** `_lyx/plan/10-teardown-skip-readings.md:54`
**Issue:** Under a covering pause the loop's watch never kills the step waiting on its agent (decision record Fact, `internal/shedverbs/doc.go`), and batten keeps its parent lock record while covered (`internal/battenshed/innerrun_parentlock.go:32-36`). "Retry once that pid ends" therefore never comes due until a resume, which the refusal does not name.

### [MEDIUM] Unknown-holder way forward is circular

**Location:** `_lyx/plan/10-teardown-skip-readings.md:54`
**Issue:** The registered driver, loop and batten of a paused run are parked, and only this removal or `reed down` ends them, so `lyx tender status` keeps listing owned processes of the slug. An unknown holder has no step record and is unregistered (`internal/pairteardown/loop.go:118-129`), so that condition does not track it either.

### [MEDIUM] Webster's doc keeps the old shell-wait rule

**Location:** `_lyx/plan/17-shuttle-shell-only-waiting.md:46`
**Issue:** `internal/websterengine/doc.go:575-580` states that no background shell expires, that every shell holds a gated turn end alike, and that `background_shell_wait_min` only sets the log threshold. Card 17 makes each of these false, and its Edit list holds only `shuttleengine` files and `docs/overview.md`. Adding the target passes `validate-plan --overlay` with no blocking finding.

### [LOW] Register's lock file is unspecified

**Location:** `_lyx/plan/02-runproc-owner-record.md:50`
**Issue:** `state.WriteJSON(path, lockPath, v)` takes a write lock at `lockPath` (`internal/state/state.go:29-41`), which leaves a file behind. The card names no lock path, so one lock file per process ever registered accumulates beside the records, and nothing says how `List` tells it apart or prunes it.

### [LOW] stopped refusal for a live pair names a retry that cannot succeed

**Location:** `_lyx/plan/11-fabric-remove-reconcile.md:35`
**Issue:** For a live branch the reconcile's way forward says there is no step now (`internal/fabricengine/originreconcile.go:320-321`). Appending "then `lyx fabric remove <slug>` again" leaves the operator with no step that makes the removal succeed. Today's busy refusal for the same run names `lyx tender pause`.

### [LOW] Owned-elsewhere survivor warning names no next step

**Location:** `_lyx/plan/13-owned-elsewhere-end.md:57`
**Issue:** With the pause standing, a survivor is left to the pair's removal or to `lyx tender reap`, but the Warn line names neither.

### [LOW] reed down help text not updated

**Location:** `_lyx/plan/12-reed-down-ends-owned.md:43`
**Issue:** `internal/reedcli/up.go:74-91` documents an unconditional kill. Card 12 lists `downCmd` as an Edit target, but its docs step names only the two `doc.go` files.

## Rejected

- The overview's Shared Decision reading D6's "ahead" as ahead or uncommitted (card 8) is not a defect. A board write leaves its change uncommitted and the sync is what commits it (`internal/boardengine/board.go:137`, `sync.go:48`). A sync deferred before its lock, as D6 requires, therefore owes an uncommitted change, and D6's "every lift spawns what they owe" licenses the reading, which the plan records.
- `--parent-locked` as a hand-typed escape (cards 6 and 7) is not a defect. The hidden `--pause-slug` flag already lets any fabric process choose its held slug, the empty one included, so the new flag widens nothing beyond its stated bound.
- The cards' order is sound: the waiting marker (5), the deferrable gate (6), `LoopSpec.OnStart` (4) and `EndOwnedElsewhere` (13) are each created no later than the first card that uses them.
