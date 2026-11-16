```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, way forward and next step the fix added can be carried out and leads to progress.

**1. MEDIUM.** D1's restore condition does not cover an in-flight strand that has not committed yet.
- Changed text (D1): "the repair may restore its last version … only when the task branch's HEAD is the end commit that version records for its last batch, so no attempt has committed since".
- Problem: an attempt of the next batch can be spawned and still working without having committed. HEAD then still equals the recorded end commit, so the restore counts as valid. But the restored state has no record of that attempt, so D12 has nothing to attach to. The next begin spawns a second agent on the same tree beside the live one.
- Evidence: the review's F4 prose, "The restore also forgets an in-flight strand, so D12 has nothing to attach to." D12's attach runs only "for every batch attempt state records in flight". The fix adds a commit check but no strand check.

**2. MEDIUM.** On a corrupt state file, `lym weaver reset` can miss the strand of the attempt in flight.
- Changed text (D2): "Over a corrupt state file it reads the run's start commit and plan commit from the state file's last committed version". The text also says the reset probes "the strand of every batch attempt and fixer state records".
- Problem: D9 commits state only after a batch ends, so the last committed version holds no record of the current batch's attempts. The probe has no output file for the in-flight attempt. The reset then moves the branch back and archives the run beside a live strand, which is the hazard F2 asked to close.
- Evidence: D9, "After each batch ends, done or failed, Go commits `_lym/weaver/`". D12, the begin and attempt identity are recorded in the live state at spawn time.

**3. MEDIUM.** After a guid-less fixer crash, "runs the verify again" may be impossible.
- Changed text (D12): "stops a strand it finds through the handle's `Stop`, counts that fixer a machinery loss (D6), and runs the verify again before any new fixer spawns".
- Problem: a fixer stopped mid-edit can leave uncommitted changes. `verifytree.Verify` refuses a dirty tree before it runs anything. The record names no outcome or way forward for that refusal, so the step stalls without reaching D6's same-rung retry.
- Evidence: PATTERN-gate-slot, Verified tree: "No site verifies a dirty tree: `Verify` refuses one with its paths before running anything."

**4. LOW.** When no state version is committed, the way forward is underspecified.
- Changed text (D2): "removing the corrupt state file and moving the task branch back by hand to the commit it held before the run's first batch commit".
- Problem: this step does not restore the plan directory in the weft. Nothing says the run then starts over cleanly. The operator must work out the start commit from `git log` alone, and the refusal does not tell them that.
- Evidence: D2's normal path restores the plan to "the weft commit the first begin records", but that begin record lives only in the corrupt, uncommitted file.

Further tool calls: none. My context already held the changed record, `attach.go`, `stop.go` and PATTERN-gate-slot.
