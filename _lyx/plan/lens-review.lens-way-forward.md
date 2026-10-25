```yaml
lens: way-forward
outcome: reported
```

Way-forward lens review of the plan's refusals, stops and recovery steps.

- **MEDIUM — D2, card 39 (adopt-refusals).** The adopt refusal's named call can be refused a second time.
  Card 39 has the refusal name `lyx adopt` with every slug both rules reach, but the card's own row says a call that names the parent is "still refused" when step 1 stops that parent as a live or diverged pair.
  So following the named call can hit a second refusal, which D2 rules out ("so following it meets no second refusal").
  When a reached parent or member would stop at step 1, the refusal must name that stop's own way forward (`--keep-local`, `--drop-local`, or ending the live driver) next to the call.
  Evidence: card 39 Intent lines 21–22 and its test row "naming the parent while step 1 stops the parent as live is still refused"; the decision record, D2.

- **MEDIUM — D8, card 18 (row-notices-after-chain).** A pause honoured during Land's wait now gets a failure notice whose way forward does not work.
  Today `landProducer.pause` blocks the row with no notice, and its `pausedReason` says `lyx batten goto <slug> --to Land` first, because shed's blocked persist leaves the pause flag set.
  Card 18 sends `RowStopNotice` for every hand-back of a Land stop whose state is not paused.
  This stop is `blocked`, so it is noticed with "fix the cause, then run `lyx batten run <slug>`".
  That run pauses again on the flag it left set, and the notice contradicts its own reason text.
  The card should exclude a pause-honoured Land stop from the notice, or name the goto first.
  Evidence: `internal/battenshed#landProducer.pause` and `#landProducer.pausedReason` (land.go), the battenshed doc's Land paragraph ("the way back is `lyx batten goto <slug> --to Land` … then `lyx batten run <slug>`"), card 18 Intent lines 23–25.

- **LOW — D8, cards 22 and 24 (shedarbiter-package, arbiterstop-batten-rule).** Batten's arbiter refusals name a loom command.
  Batten's arbiter refusals reach a batten stop report (arbiter off, spawn budget spent, and the batten eligibility refusals).
  They end with `arbiterstop.HandBackWayForward`, which names `` `lyx loom status <run-id>` shows the stop `` and shows no batten run.
  A batten run's stop is read with `lyx batten status <slug>`.
  Evidence: `internal/arbiterstop/arbiterstop.go:20`, `internal/loomcli#arbiterRefusal`, which card 22 carries over unchanged, and card 24 Intent line 18.

- **LOW — D9, card 15 (innerrun-respawn-cap).** The driver-dead notice past the respawn cap names no next step.
  Once the cap is spent, nothing respawns until the child's status file is written.
  The notice "driver strand is dead; respawned 3 times without progress" does not name `lyx loom start` in the task worktree, which brings a driver back, writes the status and resets the count.
  Evidence: card 15 Intent line 18; `#haltedWaitReason`, which already names the `lyx loom start` fallback for a halted child.

- **LOW — D4/D7, card 33 (dobby-passes).** The failing-pass notice's way forward does not move the run.
  The notice names only `lyx batten status <slug>`, a read, while dobby keeps retrying the same failure.
  It should carry the error's own way forward, or name the hand pass `lyx batten run <slug>` once the cause is fixed.
  Evidence: card 33 Intent line 31.

- **LOW — D2, cards 27 and 28 (dobby-start, batten-first-pass).** Two start reports name no next step.
  `EnsureLive` reports `prime: owned by <machine>` and does not name `lyx adopt --prime`.
  `ensureParentDobby`'s failure is only warned, with no mention of `lyx beam start`.
  Evidence: card 27 Intent line 24; card 28 Intent lines 44 and 46.

I made no further tool calls; every finding is from the cards and the exploration already in context.
