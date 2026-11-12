```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches. Three pairs break.

**Pairs checked**
- D1 / D11 (D1 says only the import reads the bolt, D11 reads it): held. The D1 criterion makes an exception for the import.
- D1 / D4 (where the maps are written): broken, see F2.
- D1 / Added-after-Discussion "D1 stands": broken, see F3.
- D2 / D7 (inherited maps at an older commit, and whether that commit is reachable from the child): held.
- D3 / D7 (after a landing the parent's pass rebuilds affected entries and leaves the rest unwritten): held.
- D4 / D5 (uncommitted maps between calls vs. the preflight exemption): held.
- D4 / D6 (only the owner commits; other records commits sweep in maps): held. A non-owner never writes a map, so no other commit there finds one.
- D4 / D7 (commit only changed maps): held.
- D4 / D9 (a task's pass commits and pushes with the code branch, and never reaches the parent): held.
- D4 / D9 / teardown (a killed task pass leaves maps that the teardown's pending-records commit takes): held.
- D4 / D11 (D4's commit takes every imported map): held. A refusal after the import leaves the maps for the next call, which D5 covers.
- D4 / Constraints `PATTERN-lym-files` (build-lock placement): broken, see F1.
- D5 / D9 (a task's hand call leaves uncommitted maps, then the run's preflight runs): held.
- D5 / D11 (an import killed partway leaves maps that are exempt): held.
- D6 / D9 (in a task, the owner is read through the parent): held.
- D6 / D11 (removal runs before the owner refusal, the import after it): held. The removal touches only untracked bolt files, so the D6 criterion "commits nothing" stands.
- D7 / D8 (an unwritten map stays byte-for-byte; the import writes compact): held.
- D7 / D11 (an imported map keeps commit `79f0eb4a` while it plans as current; `Format.Commit` still equals `Commit`): held.
- D7 / D9 / D4 (where "planned HEAD outside a pass" is read in a task): held. No reader outside a pass exists in a task; the only one is the prime's dobby.
- D9 / D10 (the dobby runs in the prime only): held.
- D9 / D11 (the operator's first hand call has no `--upkeep`, so it does not register, and a dobby pass can run at the same time): held. Both import whole files, both commit under the records write lock, and the removal waits for a committed map.
- D10 / D11 (who makes the first call, and the window until then): held. Open risks states the window.
- Added-after-Discussion "D1 stands" and "no split" / Open risks: broken, see F3.

**Findings**

- **F1 — BLOCKING, design.** Location: D4, line 50, against Constraints `PATTERN-lym-files` at line 142.
  D4 puts the map build locks "under the worktree's `.lym/testcover/`, through a scratch accessor beside `testcover.Dir`". The rule says a tracked file's lock path comes only from `lock.TrackedFileLockPath`, under the checkout's `.lym/locks/` mirror.
  The build lock locks the tracked `map.json`, and today the code builds its path with `lock.TrackedFileLockPath(MapPath, ".build.lock")`. D4's own next line routes "each map's lock" through `lock`, so the record splits one map's two locks across two schemes. The Constraints bullet carries the same split.
  Evidence: `pattern/PATTERN-lym-files.md:80-87`, `internal/gatecli/testmap.go:674-676`, `internal/testcover/coveragemap.go:210-212`.
  Fix: derive the build lock from `lock.TrackedFileLockPath` with the build suffix. Keep only the exit record on the scratch accessor.

- **F2 — LOW, consistency.** Location: D1, line 25, against D4, line 45.
  D1 says `testcover.Dir` takes the worktree's anchor and that "every reader and writer", the upkeep pass included, works through it. The anchor's `_lym` is the junction.
  D4 says the verb writes into the weft worktree's own `_lym` path, which fabric hands it, "never through the warp-side junction".
  The two decisions give the one writer two different path sources.
  Evidence: `_lym/discussion/decision-record.md:25-26,45`, and the Constraints `PATTERN-fabric-boundary` bullet at line 144.
  Fix: D1 names the readers that go through the anchor and states that the writer takes fabric's weft path.

- **F3 — MEDIUM, consistency.** Location: Open risks, lines 161-162, and D1's rationale at line 28, against both Added-after-Discussion entries at lines 128-138.
  The arbiter entries settle both Open-risks questions. They tell the fixer to drop the two lines beginning "Decision for the parent: the board entry's Idea…" and "Decision for the parent: the entry's Idea says raddle splits…", and to cite the "D1 stands" decision in D1's rationale.
  Both lines remain, and D1's rationale cites only "the operator's direction … (see Constraints)".
  Under the rubric, an Open-risks line that an Added-after-Discussion entry settles is a consistency finding, and its fix drops the line.
  Evidence: `_lym/discussion/decision-record.md:28,130,136,161-162`.

Tool calls:
1. Grepped `internal/fabricengine/doc.go` for "junction", to confirm which anchor path the junction serves. My context did not show how the anchor's `_lym` is wired.
