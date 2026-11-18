```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same maps, report files, keys or ordering.

**Pairs checked**

- D1 × D3 (the worktree map wins / the reach step refreshes worktree maps): **broken** on a dirty tree, see F1.
- D2 × D3 (changed functions read from the worktree's files / map refresh): **broken**, F1.
- D1 × D6 (only the prime's map run writes the index / the verb writes nothing): held.
- D1 × D3 run in the prime's own worktree (only the prime writes shared maps and the index / D3 builds "off the prime"): **unspecified**, F5.
- D3 × the existing redundancy step ("reuses the maps the reach step refreshed"): **broken**, F2.
- D3 × D5 (`reach.json` records changed-function hashes / the review matches HEAD): held. A note written on a dirty tree matches after the commit only if the content is identical, which is the intended behaviour.
- D3 × D7 (D3 judges nothing on a red tier / D7 runs only on a red call): held. They are disjoint, and D7 reads stored and worktree maps through D1.
- D7 × D8 (the D7 line embeds the D8 command, whose `--since` is required): held, but LOW, F6.
- D8 × D1 / D2 (old landings' changed functions / the current map's reach): held, LOW staleness, F7.
- D9 × D3 (D9 needs fresh maps / the D3 budget may fall back to the hub map): **broken**, F3.
- D11 × D3 / D9 (the hand verb `lym gate mutate` / fresh maps only built by the reach step): **broken**, F4.
- D10 × D12 (mutation needs reach / the template lists mutation in no key / the config accepts both): held.
- D10 × the redundancy step ordering: unspecified, LOW, F8.
- D4 × D5 (the capped note / the review fills from `reach.json`): held.
- D4 × D1 (own package first, specificity across maps of different ages): held.
- D7 × the failure report's 80-line cap: LOW, F9.
- Criteria and constraints: the D1 criterion ("only the prime's map run writes the index") conflicts with F5. The D3 criterion ("not rebuilt when hashes match") meets F1. The D10 criterion does not cover F3. `PATTERN-gate-slot`, `PATTERN-tolerant-records` and `PATTERN-cli-cobra` held.

**Findings**

- **F1 — BLOCKING — D2 × D3:** a writer's gate on a dirty tree cannot refresh the maps of the very packages it changed.
  - D2 diffs the merge base against the worktree's files, so uncommitted edits count as changes.
  - D3's refresh is the existing `--map` measurement. Per the `PATTERN-test-speed` background, that measurement writes no map for a package with any modified or untracked file; the package is listed as `unmeasured`.
  - Agents run their `batch` and `darn-write` calls themselves on uncommitted trees, and that is the writer-facing path the operator asked for.
  - So every changed package falls back to the stale hub map. New functions then read as "no test reaches", and the record never mentions it.
  - Fix: either run D3 only over a clean tree, or have D3's refresh measure the worktree's files (a dirty-tree map that is never written to disk). State which one in D3 and in the D3 criterion.

- **F2 — MEDIUM — D3 × the redundancy step:** "the redundancy step reuses the maps the reach step refreshed" changes what redundancy judges against.
  - Today the step judges the diff's new tests with `JudgeAgainst` against the shared map's kept tests.
  - A fresh worktree map already holds those new tests as entries. A new test could then be judged contained by itself, or by its own sibling new tests, which flips verdicts silently.
  - Fix: drop the reuse sentence, or state that the diff's own tests are excluded from the kept set.

- **F3 — MEDIUM — D9 × D3 budget:** D9 needs the changed package's fresh map, but D3 lets a package past `reach_budget_min` fall back to the hub map.
  - The hub map holds no block keys for changed blocks.
  - D9 would then report every changed block "uncovered" and plan no mutants. That is a false signal sent to review.
  - Neither D9 nor D10 says what happens to a package D3 did not refresh.
  - Fix: D10 marks such a package as not judged, with the reason.

- **F4 — MEDIUM — D11 × D3 / D9:** `lym gate mutate` runs D9 by hand, but only the `reach` step builds fresh maps.
  - The record does not say whether the verb builds them itself, under which budget, or whether it holds a slot.
  - D10's "mutation needs reach" rule does not apply to the verb.
  - D11 is the trial tool, so a verb that silently reads the hub map gives meaningless trial counts.
  - Fix: state that the verb refreshes the changed packages' worktree maps as D3 does, inside a gate slot.

- **F5 — LOW — D1 × D3 in the prime:** off the prime, `--map` writes the worktree's maps. In the prime, it writes the hub's shared maps.
  - D3's "the measurement the gate already runs off the prime" leaves a reach step run in the prime unspecified.
  - Such a run would rewrite shared maps for a partial set of packages. It is also unclear whether it writes the index, which D1 says only the prime's `--map` run does.
  - Fix: say that D3 always writes the worktree's own maps, or skips the refresh in the prime.

- **F6 — LOW — D7 × D8:** the D7 line "ends with this verb's command" but has no `--since` value to give, since no last-green record exists.
  - Fix: state that the command is printed with a `<rev>` placeholder.

- **F7 — LOW — D8 × D1:** D8 tests old landings' changed functions against the current map's reach. A function renamed or removed since then matches nothing.
  - D8's output should name the map commit, as D7's line does.

- **F8 — LOW — D10 × the redundancy step:** the order is fixed as reach → redundancy and reach → mutation, but mutation relative to redundancy is not stated.
  - The order matters only for budget and slot contention, so it is not material.

- **F9 — LOW — D7 × the failure report cap:** the report caps itself at 80 lines, and the added triage lines can push real failures past the cap.
  - Fix: state whether triage lines count toward the cap.

Further tool calls: none. Everything needed was already in context: the `PATTERN-test-speed` background, the redundancy step and report files, and the record.
