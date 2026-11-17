```yaml
lens: leanness
outcome: reported
```

Task: review the decision record and support log through the leanness lens, looking for content to move or cut.

**Findings**

1. MEDIUM — D3 Rationale. "The study measured 259 `internal/loomcli` tests as processes in 5.2 s, eight at a time" is supporting evidence. The plan writer doesn't need it to act. It belongs in the support log. The decision itself, one process per test, stands without it.

2. MEDIUM — D5 Rationale. The paragraph about what the exploration found (`tmuxkit.Main` run dirs, no fixed ports, the `.git` check) is deliberation narrative, and the same facts are already in the support log under Turn 1. Keep one clause: tests can share state outside the process, and the `.git` scratch-root check is the known cross-process collision. Cut the rest.

3. LOW — D2 Rationale. "The operator calls the current verb too complicated and ad hoc" restates the Goal and the board entry, so it can go. The smart-build citation of about 16 CPU-minutes is load-bearing for affordability, so it stays.

4. LOW — D4 Rationale. "Map mode spawns its processes from lym itself rather than through a tier shell command, so the tier's `memory_mb` scope cannot wrap them" repeats the support log's first rejected alternative. The record needs only the clamp and its bound; the "cannot wrap" reasoning can live in the support log. Keep the Bound line.

5. LOW — Constraints, the build-cache fact bullet. The 3.3 s, 3.2 s and 17.0 s / 157-package figures are more than the plan writer needs. The design rests on two facts: `set` mode reuses cached instrumented packages, and coverage builds share no cache with the normal build. Keep those with the source and Go version, and leave the numbers to the cited report.

6. LOW — Notes for the plan writer. The bullets name exact functions and files (`memoryCappedParallel`, `clampParallelToMemory`, `guardOwnTests`), and relevant.md already lists the same members. The spec allows a hint to name a symbol, so this is a duplication, not a violation. The first bullet ("move it, not write it afresh") is the useful hint; the other two repeat relevant.md.

7. LOW — Open risks, second bullet. `redundancy-sweep-existing` naming `lym gate testmap` isn't a risk this run carries. That entry already waits for this task, and its own run reads the landed code. It reads as context, so cut it or move it to the support log. If it stays, it should not imply that this run edits that entry, and it currently doesn't.

8. LOW — Scope "Out". "A map built during an ordinary gate run…" includes its reasoning, which duplicates the support log's last rejected alternative. Keep the exclusion line and drop the "since counters are process-wide…" clause.

9. LOW — Auto-mode assumptions. "The package order is import path, since the run-record ordering only served `--max`" duplicates D2, which already states import-path order. Cut it from one of the two places.

**Not a finding:** Goal, D1, D6, D7, D8 and D9 carry requirements and bounds the plan writer needs. The completeness-before-leanness test keeps them.

**Other (outside the lens):**
- D4 says each process runs under the `redundancy_test_memory_mb` cap, but D7 says nothing about a per-test cap kill as distinct from a guard kill. That may be worth a consistency glance by another lens.

No further tool calls made.
