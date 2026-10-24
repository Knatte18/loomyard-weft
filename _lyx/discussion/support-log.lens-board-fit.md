```yaml
lens: board-fit
outcome: reported
```

Lens board-fit: I checked the decision record against every other board entry for duplicated or contradicted planned work and for dependencies it leaves unnamed.

**Findings**

1. MEDIUM, Decision 8 (`carry: false`) and Scope: the record does not name `regression-baseline-verify` (F18), the note most likely to need this key.
   - F18's capture runs before implementation on an unchanged tree and writes its output to the gitignored `.scratch/numerical-baseline/`, so the tree hash does not change.
   - If F18 declares its capture as a gate tier, the green record will carry it on a second call over the same tree, and no baseline gets written.
   - F18's compare should likewise never be carried by the inert carry in Decision 5 when it is run to confirm "no drift".
   - The record introduces `carry: false` for timing only.
   - Fix: name F18 as planned work that relies on `carry: false`. A capture or compare tier must set it, or must run outside the green record. That lets F18's design pick it up rather than find out at implementation.

2. MEDIUM, Decisions 1 to 4: under `scoper: dotnet`, every tier on a `card` or `diff` key is narrowed. Nothing stops a numerical-baseline tier from being narrowed too.
   - F18 says "The compare runs at the final gate only" and asks as an open question "whether it can be scoped like the other tiers".
   - Under this record, a baseline tier listed on a `diff`-scoped key would run only the fixtures the name closure selects. That silently skips drift checks.
   - The record should say that a regression or compare tier belongs only on `all`-scoped keys, or that the scoper never narrows such a tier. It should also name F18 as depending on this choice.

3. LOW, Decision 6 (shared build) and F18's detached capture: F18 says the capture "builds with the gate properties and holds no lock that blocks the agent".
   - If the capture runs as a `lyx gate test` call that opts into `shared_build`, it runs the hub `build` step in the task worktree.
   - Webster's gates later run the same incremental build. F18 already orders its join before Webster, so `bin/` and `obj/` are not written concurrently.
   - The record could note that the build step is safe for F18's background job only because of that ordering. No contradiction today.

4. LOW, Scope, out-of-scope list: `csharp-sandbox-hub` (E28) exists to prove the C# gate path end to end, and this task is the first that needs that proof.
   - The record mentions the `cs` hub only as "writes its own config" and in a plan-writer note about its retired list-shaped `gates`.
   - It could name E28 as where the operator's first scoped run is checked, which would make the dependency explicit. The `tier1` fixture test does not replace a real hub run.

5. LOW, Scope ("Required 1 and Required 3, which task `test-tiers` delivered: the project root as a tier's `dir`, and the tier list per gate key"): the record states what `test-tiers` delivered without citing any evidence. `test-tiers` has landed and is no longer on the board, so the claim can't be checked from the board. If the Models note's Required 3 is something else, such as the slow tier at finalization, the "build only what it left out" boundary may be drawn wrong.

6. LOW, Q11 and the toolchain declaration: these are consistent with F18's operator decision of 2026-10-21, which moved tier declarations into the `test-tiers` tier config (`gate.yaml`) instead of `language.yaml`. F18's older orch recommendation still names `language.yaml`. The record could say outright that it follows the 2026-10-21 decision, which would stop a reader of F18 from seeing a contradiction.

**Checked, no conflict**

- `run-profiles-ladder`: no overlap.
- `umbrella-anchor`: a different anchor per beam leaves tier `dir` relative to the worktree root. That still works with Decision 3's whole fallback for a change outside the `dir`.
- `bench-osl1046-xdr`: no overlap.
- `adopt-runs-tests`: works with the inert carry and the build step without change.
- `pattern-restore-caller-rules`: does not touch `PATTERN-gate-slot`.
- `runs-never-write-board`: the record files nothing.
- `codex-engine`: no overlap.
- `spawn-audit-log`: no overlap. The build step spawns through the existing logged tied child.

Tool calls:
- `lyx board get` for `pattern-restore-caller-rules`, `adopt-runs-tests` and `test-time-audit-recurring`: my context held only these entries' list rows, not their bodies, and they touch gate config or PATTERN.
