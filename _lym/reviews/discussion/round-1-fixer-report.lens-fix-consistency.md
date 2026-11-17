```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fixer's change set for agreement with its surroundings, including twins left unchanged.

1. **LOW**, `_lym/discussion/support-log.md:37`, which the new D3 sentence "Map mode builds no `lym`…" reaches. The support log is stale: it still says "nothing built the prebuilt `lym` for tier 1 — fixed: D3 builds it once per call, with a criterion". D3 now says the reverse, and the criterion it mentions has been deleted. The support log is on the write surface, so its entry should record that the D3 build was reversed in round 1 (F2).

2. **LOW**, D3's new sentence, measured against D9 ("the planning, audit and lock code they reach go") and the `PATTERN-test-only-code` constraint. Neither removal list names the verb's own `lym` build, which D3 now drops. The verb's `testMapRun.buildLym` (`internal/gatecli/testmappackage.go:195,642`) is reached only from the map loop, and the record's Notes tell the plan writer to move that loop over as it stands. Nothing in the record says this build goes, so it could carry over unchanged. `prebuildLym` itself stays, because `tierrun.go:372` and `redundantmutants.go:487` still call it. The fix is for D3 or D9 to say that the measure loop's `lym` build is dropped.

I found no other disagreements. Specifically:
- D1's refusal list and D4 agree on the `memory_mb` refusal.
- The `PATTERN-refusal-way-forward` constraint names D4.
- The Scope list, the `PATTERN-gate-slot` constraint and the Review criterion all name `pattern/PATTERN-gate-slot.md`.
- D6's new ownership rule matches its criterion, and D7 matches D6.
- D9's widened refusal set matches the Scope line, and D9's criterion covers the failed-build-only case.
- D4's Bound naming tier 0's `parallel` matches `testmappackage.go:604-608`.

Calls made:
- grep of `support-log.md` for prebuilt, D6 and D9 phrasing, plus a grep for `buildLym` and `prebuildLym` callers. My context did not show the support log or who calls the `lym` build.
