```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism on one concrete case and checked the end state against the decision's purpose.

**Findings**

1. MEDIUM — D1, detached path. A map call that goes through the detached runner loses `--map`, or joins an ordinary run.
   - Evidence: `cappedCall` hashes `request.words()` into the detach key dir (`detachKeyDir`), and the runner replays those words. `words()` in `internal/gatecli/detachfront.go` rebuilds the call from an explicit field list: dir, gate, tier, upto, card, main-packages, base, all, …
   - Case: an agent strand runs `lym gate test --map internal/x` with no `--wait`. The front builds words without a map field. It joins or starts the key of `lym gate test internal/x`, and the runner runs an ordinary tier 0 call. No map is written, and the result is an ordinary green or red.
   - The purpose of D1 ("agent calls … go through the detached runner like any other gate call") is missed. The record does not say that the flag must be part of the replayed words and of the join key.
   - Fix: D1 states that the detached call's words and key carry `--map`, so a map call never joins an ordinary run. Add an acceptance bullet for an agent-strand `--map` call.

2. MEDIUM — D4, memory clamp with `memory_mb` but no cgroup limit.
   - Case: tier 0 has `parallel` 12 and `memory_mb` 8192, the per-test cap is 2048, the cap covers the tree, and lym runs in an unlimited cgroup (`limited` false).
   - `clampParallelToMemory` as it stands returns `parallel` (12) on `!limited` before it looks at headroom. D4's "headroom is also at most that value" only binds if the tier cap also counts as a limit. Read literally, the existing function leaves 12 × 2048 MB, which is more than the 8192 MB tier cap.
   - D4's purpose (the tier's cap bounds the run's processes together) is met only if the decision says `memory_mb` counts as a limit when no cgroup limit exists. The record says only "extends".
   - Fix: state it in D4: with `memory_mb` set, side-by-side ≤ `memory_mb` / per-test cap, whether or not lym's cgroup is limited.

3. LOW — D3/D7, prebuilt `lym` for tier 1 tests.
   - D3 says tier 1 tests run "with the gate's prebuilt `lym`". The gate builds `lym` once only in the shared build step, inside the first tier that opts in. D7 does not say map mode runs that step, and the verb today has its own `buildLym`.
   - Case: a hub whose tier 1 does not opt into the build step. The prebuilt binary is never produced, so a tier 1 test reaching `lymbin` builds its own `lym` per process.
   - That is a cost, not a wrong map, but the step the decision relies on is never given its input. Fix: D3 or D7 states that map mode builds `lym` once per call and exports it.

4. LOW — D6, the dirty check covers Go files only.
   - Case: a modified embedded stencil `.md` or a `testdata` file in a mapped package. The tree passes D6's "modified or untracked Go file" check, and the tests measure the dirty content. The map is stamped with HEAD's commit.
   - That is the false stamp D6's rationale says the refusal prevents. Fix: refuse any modified or untracked path under the module, or say the stamp covers only Go files.

5. LOW — D7, a failed build drops old entries.
   - D7 says a failed build "keeps no old entries", a change from today, where a failed build keeps the map's entries.
   - Case: a package whose build fails at one commit. After this change its map is empty, and `lym gate redundant <pkg>` judges nothing until a rebuild. The record does not say how `lym gate redundant` reads a build-failure map. That is consistent with D2's whole-rebuild purpose, but the change of what consumers see is not named in D8 ("nothing in this task changes what they read").

**Decisions walked**

- D1 — Case: `lym gate test --map internal/x` from an agent strand without `--wait`. Purpose missed on the detached path (finding 1); met with `--wait`.
- D2 — Case: an existing map of `internal/x` with an entry for a test since deleted. A whole rebuild replaces the map, so the entry is gone. Met.
- D3 — Case: `internal/loomcli`, one coverage binary, a `-test.run` per test, one profile each, merged and written at package end. Met, except the tier 1 `lym` input (finding 3).
- D4 — Case: `parallel` 12, `memory_mb` 8192, per-test cap 2048, unlimited cgroup. Not met as worded (finding 2). The guard via `guardOwnTests` holds: an enclosing `LYM_GATE_TEST_PID` is set only in a tier's env, and map mode has no outer tier.
- D5 — Case: two processes of one package, one tripping `tmuxkit.Main`'s shared-root `.git` check. The failure reruns alone, passes, and is listed. Met.
- D6 — Case: a modified `.go` file is refused (met); a modified embedded `.md` is not (finding 4). A HEAD move before a write leaves the package unwritten and the call exits non-zero (met).
- D7 — Case: a run with one test failing alone. The map is written with the failed entry and the exit is non-zero. No green record is written, so the next call runs its tiers. Met; the failed-build entry drop is noted (finding 5).
- D8 — Case: two map runs over the same package from one worktree. Each write goes through `internal/state` and the later one wins with a whole map. Met.
- D9 — Case: `lym gate redundant internal/x` with no map. The refusal names `lym gate test --map internal/x`. Met.

**Tool calls**

- Grepped `LYM_GATE_TEST_PID` and the detached front's join and key code: my context lacked how the guard's ancestor marker is set and how a detached call is keyed and joined.
- Read `joinState`, `cappedCall` and `frontDecision` in `internal/gatecli/detachfront.go`: my context lacked which call fields form the detached key.
- Read `testRequest.words` and `detachKeyDir`: my context lacked whether the key and the replayed words come from an explicit field list that a new flag must join.
