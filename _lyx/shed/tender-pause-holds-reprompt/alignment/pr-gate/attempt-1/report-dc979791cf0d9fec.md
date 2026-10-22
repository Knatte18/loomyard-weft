---
verdict: APPROVED
---

## Alignment at PR-Gate: `tender-pause-holds-reprompt`, PR 821 at `fa94adee5`

### Goal and decisions against the diff

The board entry (issue #762 plus the 2026-10-21 webster note) asks for four things: a paused scope gets no new prompt, a verify command running in a paused scope is stopped or held, `lyx creel interrupt` works on a covered session, and `lyx webster begin-batch` refuses under a pause.
The diff covers all four, along the lines of decisions D1–D8:

- D1/D2: `internal/shuttleengine/pausedsend.go` adds `ErrPaused`/`PausedError`, and each send context gets a `paused` check.
  That check judges the target by its strand name through `agentname`, falling back to the runner's slug.
  The tender and creel runners are told the hub record with an empty slug (`tendercli/deps.go`, `creelcli/delivery.go`).
  `orchcli` gets no `SetPause`, so the orch runner reads uncovered, and `orchcli/pausegeometry_test.go` pins that.
  The callers (seat notices, creel delivery, tender resume claim, `lyx shuttle send`, loom driver resume) are changed in their own packages.
- D3: the gate loop's boundary hold is in `shuttleengine/wait.go` and `run.go`.
- D4: `verifytree/hold.go` and the `verify.go` hold handle the wait before the command, the stop after the grace end and the `paused` marker.
  `proc` gets a new SIGTERM-first group cancel with its own SIGKILL timer and group-empty poll, used only by `verifyrun`.
  `ConfigureGroupKill` is unchanged.
  `loomcli/reviewstatus.go` and `battencli` read the paused marker.
- D5: every verify site passes a hold: loomshed, the webster gate (on loom runs too, after Merge-Review fix F1 in `loomcli/wiring.go`), landingshed's Merge-In/Finalize/Publish, and `lyx webster verify`.
  The landing hold reads hub entries only while `parentlock.Store.HeldBy` reports the run's lock.
  The `cmd/lyx/verifiedtree_test.go` tripwire now checks for a `Hold`.
- D6: `pauserecord.ClaimSession`.
- D7: `creelengine/verbs.go` claims the covered strand, then interrupts it, re-reading that the entry is still live before sending the key.
- D8: `webstercli/batchpause.go` refuses `begin-batch`/`recover-batch` with the two refusal texts.
  Neither text names `lyx resume`, and the verb claims Master through the strand GUID.

What the decision record puts out of scope stays out: `cardverify` and batten's `verifyrun` rows get no hold, `lyx tender pause` is unchanged, and processes other than verify commands are left to `run-process-ownership`.

### PATTERN

The two declared changes are present: the new `PATTERN-pause-holds-prompts` with its background file, and `PATTERN-verified-tree`'s line and background.
The declared text of the new entry exceeds `pattern.MaxEntryLineChars`, so the overview line is shortened and the full text sits in the background, as the plan's overview says.
Plan card 8 also trims eight overview lines to stay under `pattern.HeadroomBytes`.
The set differs from the card's list after the merge of `main`, which Merge-Review's F4 commit settled.
I checked every cut clause, and each still appears in its entry's background file: cwd-resolution, fabric-vocabulary, mutation-record, gogit-read-helper, review-round, friction-capture, hubforge-fixtures and documentation-lifecycle.
The overview ends 25 bytes shorter than on `main`.
These trims change no rule, so they are bookkeeping that the headroom forces, not a scope deviation.
This is the substance of Plan-Review's carried-over `undeclared-pattern-cuts-unrecorded`.

### Carried-over open risks

- Merge-Review's BLOCKING `loom-webster-gate-unheld` is fixed by `070547355`, with its seam-list follow-up in `fa94adee5`.
- Merge-Review's `webster-gate-hold-untested` is fixed by `fb3d4bb92`.
- The rest are residuals that the decision record's `## Open risks` already accepts and that no recorded decision is broken by:
  - Webster-Review's `boundary-held-survives-turn-change`;
  - Discussion-Review's `d4-sigterm-cancel-home-unstated`, which D4's text now settles by placing the cancel in `internal/proc`'s platform files;
  - the D7 resume window;
  - the deferred-submit window;
  - the failed-GUID Master wake;
  - the Windows group kill, which the `verify-watchdog` note owns.

### Board

- `run-process-ownership` depends on this task and owns the remaining pause leaks: driver chains, parked pushes and background work.
  This run does nothing from that entry.
- `batten-cancel-verb` and `verify-watchdog` overlap only by topic.
  This run leaves the Windows tree kill to `verify-watchdog`, as its decision record says.
- No entry is made obsolete by this run.

### Overlaps

`lyx loom peers` lists no parent commits and one peer.

- `machine-handover` (row 16; own row 21) shares these files: `PATTERN.md`, `internal/pauserecord/{doc,record,record_test}.go`, `internal/tenderengine/{doc.go,resume.go,pause_test.go}`, `internal/tendercli/{deps.go,cli_test.go}`, `internal/loomcli/{landingdeps,landingdeps_test}.go`, `internal/landingshed/deps.go`, `internal/battencli/{wire,wire_seams_test}.go` and `internal/testkit/envkit/envkit.go`.
  Its pause changes are additive: an `OwnerCaller` label, a `Coverage.OwnerHeld` field and an owners section in tender status.
  Its landing deps add owner seams.
  None of these changes the behaviour this run changes.
  Its decision record names this task as the owner of the pause's verify re-prompt leak, so this run carries out that handoff rather than contradicting it.
  The peer's D6 lets a run holding the parent lock skip slug pause entries during Finalize.
  This run's landing verify hold reads hub entries only while that same lock is held, which is the same exemption, not a conflict.
  In the PATTERN overview, the peer adds `PATTERN-owner-machine` and this run adds `PATTERN-pause-holds-prompts`, with headroom trims.
  Those are different entries, so it is a text overlap only, though the peer must keep the overview under headroom at its Merge-In.
  Landing order: this run is further along (row 21 against 16), so it lands first, and `machine-handover` takes these changes at Merge-In.
