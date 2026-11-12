```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on a concrete case and checked whether it meets its purpose.

**Findings**

- **MEDIUM — D6 and D9 together: maps never reach the compact form.**
  - Case: D9 copies the 211 indented maps across unchanged and commits them.
  - D6 then leaves every current map's bytes untouched, so a map stays indented until a landing affects it.
  - Packages no landing touches stay indented indefinitely, so D2's "about 41 MB in the compact form" per weft checkout does not hold. It stays near 56 MB for a long time.
  - Evidence: D6 skips any map that is current with no dropped or queued entry. D9 says "copies every `map.json` across".
  - Fix: D9 rewrites each imported map in compact form before its commit. Alternatively, D2 states the size honestly.

- **MEDIUM — D4 against the Constraints line on `PATTERN-fabric-boundary`.**
  - The Constraints line says "code that writes the maps through the junction never commits them".
  - D4 has the same pass write the maps through `<anchor>/_lym` and then commit them.
  - The step's input does not match what the constraint claims. The pass writes through the warp-side `_lym` link and is also the committer.
  - The PATTERN rule is "Code that writes into `_lym` through the junction never commits it; the weft commit is Go calling the engine at a boundary the run controls". The upkeep pass is not a run.
  - Fix: either say the pass writes into the weft worktree's path directly, as `fabricengine.WeftLymDir` gives it, and commits at its own end boundary, or name this as a departure from the entry.

- **LOW — D5: the owner rule does not stop two machines when no prime owner is recorded.**
  - Case: a hub with no `prime_owner`. `PATTERN-owner-machine` says such a root "reads as owned by no other machine".
  - So both machines pass the D5 check, and both commit the prime's maps. D5's purpose, "two machines never commit one branch's maps", fails.
  - Fix: state this as a bound, or refuse when no owner is recorded.

- **LOW — D4: a killed pass's maps may never be committed.**
  - Case: a killed pass while `test_map_upkeep` is off (D8). There may be no "next pass", so the written maps sit uncommitted in the prime's weft worktree.
  - A child forked meanwhile does not inherit them (D2).
  - Fix: the acceptance criterion holds only once a pass runs; state that as a bound.

**Per decision: the case used, and whether the purpose is met**

- **D1 — met.** Case: a task gate reads `<task anchor>/_lym/testcover/<pkg>/map.json`, which was inherited at fork.
- **D2 — met for the files, not met for the size claim.** Case: a pair is added from the prime after D9. Its weft fork keeps `testcover`. Size: see the first finding.
- **D3 — met.** Case: task A changes `internal/x`, lands, and is archived. The prime's pass plans `x` at the new HEAD and drops the affected entries. A's map file never reaches the prime's weft.
- **D4 — met, with the two findings above.** Case: a prime pass measures 3 packages, then makes a scoped commit and a records-only push.
- **D5 — met where an owner is recorded; not met where none is.** Case: the second machine runs `lym gate testmap` on the prime.
- **D6 — met.** Case: a landing touches only `internal/a`, and package `b` is current. `b`'s file keeps its old `commit`.
  - The next selection diffs from `m.Commit` with the recorded `m.Files` hashes, as `testselect.go` lines 180–200 and 266–318 show.
  - `Format.Commit` and `BuiltAt` stay equal to the map's, since nothing rewrote it, so there is no `MissUnmarked` (line 224).
  - So the claim "the selector already diffs from the map's own commit" holds.
- **D7 — met.** Case: an agent runs `lym gate testmap` in a task worktree at the task's HEAD. It writes the task's maps and its commit pushes the code branch too.
- **D8 — met.** Case: a hub with no key. Its dobby starts no pass.
- **D9 — met for the import; the compact form is not reached.** Case: the prime is empty and the bolt holds 211 maps. The pass imports them, plans, measures up to the cap, commits, and removes the bolt directory. A second run finds maps and imports nothing.

**Tool calls**

- grep in `internal/testcover/testselect.go` for `m.Commit` and `Files`: my context lacked how the selector uses a map's commit, which D6's claim depends on.
- sed of `testselect.go` lines 180–232: my context lacked the format-marker check and the recorded-hash comparison, needed to confirm that an unrewritten map is still read as current.
