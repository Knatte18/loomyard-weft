```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the amended board entry's goal points and checked every goal point for a change that builds it.

**Decisions** (decision record alone: goal points under `## Goal`, plus the `Added after Discussion` entries)
- G1, from #285: before writing a fix, the review checks that any test infrastructure the fix names exists: a kit, a recording, a stand-in, or a tier able to run it. When it is missing, the finding says so and offers a fix that works without it, or is raised as a `decision` finding: accept the gap, or build what is missing.
- G2, from #288: the review stencils check a fix against the package clause and export status of any identifier it reuses, and against the callee chain behind any claim that code only reads. A fact the reviewer could not establish is marked unverified in the `fix`.
- G3, from #291: plan review checks each card's claims about readers, visibility and reachability against the code, and checks the cards against each other.
- G4, from #290: plan writing and plan review keep a new production function in the same gated unit as its first caller, or give cards a way to carry and retire a temporary allowlist entry.
- A1, the `Added after Discussion` entry "No test seam": the red-first check does not apply, and no test asserts stencil prose.

**Changes and what they trace to**
- C1, `burler-review-format.md:33-36`: the three fact checks before a `fix` (identifier visibility, callee chain, test infrastructure). Traces to G2 (first two) and G1 (third).
- C2, `burler-review-format.md:37-39`: never write a disproved fix; propose one that holds without the fact, or raise class `decision` with the two choices. Traces to G1.
- C3, `burler-review-format.md:40`: an unestablished fact is named in the `fix` as unverified. Traces to G2.
- C4, `burler-step-fix.md:29-30`: the fixer reads a fact named as unverified before applying the fix. Forced by C3, since the fixer is the only consumer of that marking.
- C5, `loom-block-plan-cards.md:60-64`: the plan writer reads card claims in the code and checks cards against each other. Traces to G3. The entry names plan review only, but the rubric's writer/reviewer symmetry note at `loom-rubric-plan-review.md:146-148` forces the writer side.
- C6, `loom-rubric-plan-review.md:63-71`: the "Card claims against the code" and "Cards against each other" bullets. Traces to G3.
- C7, the first-caller rule at `loom-block-plan-cards.md:56-58` and `loom-rubric-plan-review.md:58-62`. Traces to G4, taking its first alternative.
- C8, `loom-plan-spec.md:366-368`: the granularity exception. Forced by C7, because the spec's existing public-surface rule at `loom-plan-spec.md:365` would otherwise contradict it.

**Per decision**
- G1 is built by C1 and C2.
- G2 is built by C1, C3 and C4.
- G3 is built by C5 and C6.
- G4 is built by C7 and C8.
- A1 is honoured: the diff adds no test.

**Findings**: none within this lens. Every change traces to a goal point or is forced by one, and every goal point has a change that builds it.

One point outside this lens, left to the affected-set lens and the handler: G4's premise is that the card gate runs the reachability guard. It does not. The guard carries `//lym:only upkeep` (`cmd/lym/testonlycode_linux_test.go:91`) since commit `2326ef4da`, which is an ancestor of the review base. `pattern/PATTERN-test-only-code.md:30-32` states "A card may therefore add a function that only a later card calls." C7 and C8 contradict that line.

Tool calls: none. My context already held the diff, the decision record, the stencils and the pattern background.
