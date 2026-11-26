```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the record's bounds, guarantees and claims about existing code with concrete counterexamples.

**Findings**

1. **BLOCKING, D12 and its tier0 import-guard criterion.** The closed test-import set D12 allows would refuse imports fabric's own tests need today, and the test PATTERNs mandate some of them.
   - Evidence: a count of `internal/*` imports across `internal/fabricengine/*_test.go` shows these, none of them in D12's set:
     - `internal/lymcwd` (84)
     - `internal/lymdirs` (26)
     - `internal/fslink` (25)
     - `internal/gitexec` (20)
     - `internal/lock` (18)
     - `internal/gitrepo` (18)
     - `internal/weftname` (11)
     - `internal/testkit/boardkit` (11)
     - `internal/testkit/logcapture` (10)
     - `internal/testkit/permkit`, `internal/testkit/locationkit`, `internal/runproc`, `internal/proc`, `internal/logger`
     - `internal/dotgit`, `internal/state`, `internal/parentlock`, `internal/testkit/takeoverkit`, `internal/testkit/scankit`
   - Counterexamples:
     - `fabricengine.ReconcileOrigin` takes a `*lymcwd.Location`, so a seam test for it cannot be written without `lymcwd`.
     - `PATTERN-test-isolation` mandates `permkit.Chmod` for read-only directories.
     - Tests of the lock and TOCTOU behaviour, which D4 keeps in the engine, need `lock` and `parentlock`.
     - Tier1 action tests asserting git effects use `gitrepo` and `gitexec`.
   - Fix: define the set as fabric's own packages, the leaf and infrastructure packages fabric's production code imports, and the mandated kits. Exclude only feature modules: `shedrun`, `shedengine`, `boardengine`, `fabriccli`, `heddleengine`, `stencilstore`, `hubreconcile`, `vscode`, `configengine` and `yamlengine`.

2. **MEDIUM, D2's lock bound.** "The reader runs under the lock the verb takes today, at the point it takes it" does not hold for verbs that refuse before any lock.
   - `PATTERN-fabric-boundary` says `Commit` refuses a secret-path add "before any lock".
   - It also says `Topology.Add` writes gc keys "once its pre-flight refusals pass".
   - So these verbs have a pre-lock read and a post-lock read. A single snapshot taken at the lock point either moves the pre-flight refusal under the lock, which changes timing and lock hold time against D5, or needs two snapshot phases.
   - The record should say a verb may have a pre-lock and an under-lock snapshot.

3. **MEDIUM, D2's "no more git processes than the verb's current reads".** Today's verbs read lazily and short-circuit.
   - Counterexample: a remove that refuses on a dirty worktree never runs `ls-remote` today.
   - A per-verb reader that fills every field the verb's rules use, origin included, runs `ls-remote` on that refusal path too.
   - D3's re-read loop adds reads per round.
   - The bound holds only if readers are staged by rule phase or read lazily, and the record should say which.

4. **MEDIUM, D3 and D4 against detached children.** The record says the engine runs every action in-process. Fabric's ordering promises run in detached children instead:
   - `SpawnPairPush`, in `spawn.go`.
   - The teardown child that pushes `archive/<slug>/*` tags before any delete on origin.
   - A child is a separate lym process with its own reads and decisions, mid-sequence and after the parent has exited.
   - The record does not say whether a child takes an action-list tail from the parent or re-snapshots and asks the core itself.
   - It also does not say how the archive-tag-before-delete order (`PATTERN-fabric-boundary`) holds across that hop.

5. **MEDIUM, D10's target can be met by moving cost.** D12 moves cross-module tests that build hubs into `shedrun`, `shedengine`, `boardengine` and other packages, often still through `hubforge`.
   - That lowers the fabric packages' tier1 time without lowering the tier1 run.
   - Counterexample: about 25 cross-module files move with their hubs, and the 60 s target is met while the 250 s run stays about the same.
   - D10 should also record the whole tier1 run's time and the moved tests' cost in the packages they moved to.

6. **LOW, D4's claim that `Mutations` "stays the record of executed effects".** It holds, but the PATTERN text it sits beside does not.
   - `PATTERN-fabric-boundary` says "The `rec *Mutations` recorder is threaded into `destroy.go` only."
   - Yet `*Mutations` already appears in `add.go`, `cleanup.go`, `anchorchange.go`, `merge.go`, `pushclassified.go`, `originreconcile.go` and about 14 more production files.
   - D4 makes the executor the recorder for every action. The `PATTERN-fabric-boundary` change should restate that clause, or the plan writer gets a rule that contradicts D4.

7. **LOW, D1's gate-scope rationale.** "Keeps the hundred-odd importers of fabric's names out of the gate scope of a rule change" overstates the gain.
   - `internal/fabricengine` imports `internal/fabriccore`, so a rule change still pulls in about 50 production importers of fabricengine and their test importers.
   - Only importers that need nothing but layout names drop out.
   - The claim should be narrowed to that.

8. **LOW, D1's closed import sets.** "Leaf packages only" may break `internal/fabriclayout`'s pure path functions.
   - If they take a `*lymcwd.Location` the way the engine's API does, the layout must import `lymcwd`.
   - `lymcwd` imports `configengine`, `logger` and `weftname`, so it is not a closed leaf of the kind D1 implies.
   - The record should say layout path functions take absolute paths and strings, which `PATTERN-cwd-resolution` also requires: "engines get absolute paths, never import lymcwd".

9. **LOW, Open risks and Turn 1's "No peer run changes `internal/fabricengine`".** That holds for fabricengine itself, but peer `pr-modes-no-agent` changes `internal/testkit/envkit/envkit.go`, which D7 retargets. It is only a text overlap, and that run lands first, so this is no conflict, just a note.

**Claims attacked**

| Claim | Result |
|---|---|
| D1: core runs no git and does no I/O; leaf sets | Held, except layout path functions (finding 8) |
| D1: gate-scope rationale | Overstated (finding 7) |
| D2: snapshot read under the verb's lock, at its point | Broken for pre-lock refusals (finding 2) |
| D2: no more git processes than today | Broken on short-circuit paths (finding 3) |
| D2: `dotgit.ListWorktrees` refuses non-`files` ref storage, fallback to git | Held (pinned in `internal/dotgit/doc.go`) |
| D3: loop never runs more rounds than today | Held as a bound, if the core keeps today's classes |
| D3 and D4: engine runs all actions in-process | Broken by detached children (finding 4) |
| D4: destructive actions only through `destroy.go`, checks at execution | Held |
| D4: `Mutations` as the record | Held, but PATTERN text is stale (finding 6) |
| D5: tests deleted only with their replacement | Held, as a review check |
| D6: interleaving tests stay tier1 or tier2 | Held, but they need imports D12 bars (finding 1) |
| D7: retarget in one foundation change, no aliases | Held |
| D9: serial order satisfies `packages-shared` | Held |
| D10: 60 s target shows the gain | Gameable (finding 5) |
| D12: closed test-import set | Broken (finding 1) |
| Constraints: gate-scope fact | Held, per the research |
| Peers: no overlap | Held for fabricengine; envkit text overlap (finding 9) |

**Tool calls**

- Counted the `internal/*` imports across fabricengine's test files, to test D12's closed set. My context had only the parent's relayed module list.
- Grepped `*Mutations` users, `lymcwd`'s leaf status and the detached-spawn files, to test D4, D1 and D3. My context had none of these.
