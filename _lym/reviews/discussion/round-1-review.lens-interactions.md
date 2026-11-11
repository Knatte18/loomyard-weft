```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions, and decisions against criteria and constraints, for one undoing or contradicting another.

**Pairs checked**
- D1 × D4 (drops and gaps vs requeue): broken, see F2.
- D1 × D5 (Affected-verdict write at H, then a failed build): held. The failure field names H, and the Commit is already H.
- D1 × D6 (re-stamp vs stamp): held. A Current verdict implies an equal stamp.
- D1 × D7 (kept entries vs self-check): broken on wording, see F6.
- D1 × D8 (lock-held skip vs the next start): broken, see F5.
- D1 × D10 (plan write vs per-round job writes under the build lock): held.
- D2 × D3 × criterion D2 (testdata edit is a miss): broken, see F4.
- D2 × D6 (stamp miss): held.
- D4 × D2 (outcome entry is no recorded test, gate runs it): held.
- D4 × D7 (audit covers passing entries only): held. The bound is stated in D4.
- D4 × D8 (spawn-marked tests out of the queue): held.
- D4 × D9 × D10 (killed run queued again): held within one pass, broken across passes, see F2.
- D5 × D8 (failed-at-HEAD carve-out vs the failure-only `BuildFailure` map): unclear, see F7.
- D8 × D9 (start rule vs a hub with fewer than two slots): broken, see F3.
- D8 × D9 (dobby pass vs hand call: which call is which): broken, see F8.
- D8 × D11 (anchor move ends the pass, next start): held.
- D8 × D12 (the "names queued work" trigger vs the exit record's queued count): unclear, see F7.
- D9 × D10 (yield at round and poll, finished entries written): held.
- D9 × D7 (audit takes its slot by D9's rule): held.
- D10 × D8 (HEAD read per round and per job): held.
- D11 × D10 (signal writes the finished rounds): held.
- PATTERN-config-layers Constraint × D8 reinterpretation: broken, see F1.
- PATTERN-gate-slot Constraint × D1/D9 (the plan step holds no slot): held.
- Criteria for D1, D3, D6, D10, D11 and D12 against their decisions: held. Criteria for D2 (testdata), D4 (yield requeue) and D8 (start on queued work) are broken, see F2–F4.

**Findings**

F1. BLOCKING, design. Location: Constraints › PATTERN-config-layers × D8.
- Summary: The Constraint retires `test_map_every` as a retired key, so the reconcile walk drops it from the hub base file. The walk does that before it applies reinterpretations, so D8's reinterpretation never sees `test_map_every: 0`. A hub that switched maps off silently gets upkeep on. D8's criterion ("reads as `test_map_upkeep: false` after the reconcile walk") then fails.
- Evidence:
  - `internal/hubreconcile/walk.go:232`: `configsync.ReconcileHubWideAt` runs first.
  - `internal/configsync/configsync.go:1157` (`pruneSparse`), with `:719`/`:727` (`DropRetired`): units a retired key covers are removed even without `PruneUnknown`.
  - `internal/hubreconcile/walk.go:254`: `applyReinterpretations` runs only after that.
  - `configreg.Module.Migrate` (`internal/configreg/configreg.go:77-79`) runs before the prune (`configsync.go:1141`) and exists to rewrite retired keys. A reinterpretation does not fit this case.

F2. BLOCKING, design. Location: D1 × D4 × D8 (and the D4 criterion "a test killed by a yield … is queued again").
- Summary: D1 queues work only on an Affected or Miss verdict. A gap left at a map whose `Commit` is already H is never queued again: a test dropped by the plan whose job then ended, a killed or discarded round, or a signal. The next plan at the same or a later HEAD reads Current, because nothing differs, so the gap stays until an unrelated change.
- The same holds for an outcome entry dropped on an own-file change under a Current verdict (D4). The test is dropped and nothing queues it.
- So D8's restart on "queued work" re-plans and finds nothing, and the D4 criterion holds only inside one pass. D8's "a backlog drains on its own" is false for these gaps.
- Evidence: decision-record.md D1 lines 34–40, D4 lines 92–94, D8 line 142, D10 line 184, criterion line 273.

F3. MEDIUM, design. Location: D8 × D9 (one-slot hub).
- Summary: On a hub with fewer than two slots, D9 has a pass plan, re-stamp and end with its queue untouched. The exit record then names queued work, and D8 starts a new pass on every dobby tick. That restart loop is the one D8's carve-outs exist to prevent: dirty, failed-at-HEAD and spawn-marked work are excluded, but "no slot can be taken" is not.
- D9's Bound ("only hand calls build maps") does not state the churn.
- Evidence: decision-record.md lines 141–147 and 169, 176. `slots` is `CategoryMachine` (`internal/gateslot/config.go:148`), so a machine layer can lower it.

F4. MEDIUM, consistency. Location: D2/D3 × criterion D2 ("an edit to a file under B's `testdata` is a miss").
- Summary: D3 records the own package's non-Go file hashes "by the rule it uses for a linked package's". That rule, per `testmap-cross-package`'s Delivers, covers the directory's files plus `go:embed` targets in subdirectories. A `testdata/` file that is not embedded is then never hashed, so its edit is no miss, which contradicts the criterion.
- The record must state that the own-package set includes `testdata/`, or the criterion is wrong.
- Evidence: decision-record.md lines 62–63, 83, 270. Board entry `testmap-cross-package` Delivers: "per-linked-package file hashes (go:embed targets in subdirectories included)".

F5. LOW, consistency. Location: D1 × D8.
- Summary: D1 says a map skipped for a held lock is planned again by "the next pass". No condition in D8 starts a pass for it: HEAD is unchanged, and D12's record has no lock-skip count. Unless lock-skipped maps count as queued work, they wait for the next landing.
- Evidence: decision-record.md lines 44–45, 141–142, 203–207.

F6. LOW, consistency. Location: D3 × D7.
- Summary: D7's self-check requires each kept entry's "linked functions" to "appear in H's block table". D3 stores no block of another package, so the map's block table cannot hold them. The check needs the linked function set of the fresh build, not the stored table.
- Evidence: decision-record.md lines 81–82, 128.

F7. LOW, consistency. Location: D5 × D8 × D12.
- Summary: Two points are unclear.
  - D8's carve-out names "a package whose recorded failure names the current HEAD". It does not say this covers D5's failure-only map (`BuildFailure` plus `Commit`) as well as the new failure field. If it does not, a failing first build restarts passes each tick.
  - D12's "queued package count" does not say whether it is the plan's count or the work left at exit. D8's trigger "that record names queued work" needs the latter.
- Evidence: decision-record.md lines 106–108, 146, 204.

F8. LOW, consistency. Location: D8 × D9.
- Summary: D8 splits behaviour between "a pass the dobby starts" and "a hand call". D9 defines only a hand call "naming packages". With `test_map_batch` (`--max`) retired, the record never says how the verb tells a dobby pass from a hand call that names no packages. So it is open which rules apply to such a call: the exit record write, the non-blocking take with yield, or the blocking take.
- Evidence: decision-record.md lines 148–150, 170–171; `internal/beamcli/dobbydeps.go` `startTestMap` passes only `--max`.

Tool calls: none.
