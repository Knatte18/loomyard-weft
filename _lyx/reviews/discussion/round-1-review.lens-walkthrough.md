```yaml
lens: walkthrough
outcome: reported
```

Task: the walkthrough-lens review of the decision record. I ran each decision's mechanism end to end on one concrete case.

**List: each decision, the case used, and whether its purpose is met**

- **D1** — met. A hub sets `enabled: true` and declares `capture` and `compare`, each holding `{baseline_dir}`. The commands render the lyx-owned directory, and an undeclared command is refused, naming `lyx config drift`.
- **D2** — met. With the flag off, a loom run records `Drift-Baseline` Skipped. `Drift-Await` finds no scope record and returns Done, and `Drift-Verify` is Skipped. Darn's order runs Preflight → `Drift-Baseline` → Darn-Write, with no await row.
- **D3** — met. Loom, scope `partial --skip A`: the agent's `lyx drift scope` writes `scope.json`. The row starts the detached capture, returns Done once the child holds the lock, and Plan-Write runs meanwhile.
- **D3/D5 re-capture** — not met at acceptable cost. The capture fails, `Drift-Await` is Stuck, and its named way forward is `goto Drift-Baseline`. The recapture then returns Done onto `on_done: Plan-Write`, which fresh-spawns and rotates away the approved plan. The whole Plan perch reruns.
- **D4** — not met across an adopt. The capture passes and `awaited_at` is set, then the run is adopted to machine B. B has no `.lyx/drift/` (no capture record, no baseline), so `Drift-Verify` is Stuck. The purpose, a compare against the branch-point baseline, is lost.
- **D5, way forward `goto Webster`** — not met. After a failed capture, the operator picks `goto Webster` "to run without regression". `Drift-Verify` then halts again, because the scope has no passed capture record. Each Webster reroute through the plan perch also halts again at `Drift-Await`, since `awaited_at` was never set.
- **D6, scope `full`, acceptance** — not met. The agent accepts T and the compare passes. The row is Stuck naming "`lyx drift accept` and resume, or fix and resume". On resume the accepted list is still non-empty, so the row is Stuck again. No named way forward reaches Done.
- **D6, budget** — not met. The row halts past `verify_max_runs` and is resumed. The record never says when the Go-written counter resets, so the next agent session either has no budget at all or an unbounded one.
- **D7** — met within one session. The agent's `lyx drift accept T` adds T to the accepted list, and the next compare's `{skip_file}` lists T.

**Findings**

1. MEDIUM, design. D5's way forward `lyx loom goto Drift-Baseline`, which D3 says "captures again", re-enters through `Drift-Baseline`'s `on_done: Plan-Write`. Plan-Write's fresh spawn runs `NewPlanDirRotator`, which archives the approved plan, and the full Plan perch reruns. The record never states that cost, and it never bounds the recapture to skip the replan. Evidence: `decision-record.md:118`, `:82-84`, `:53`; `internal/loomshed/planwrite.go:96-100`; `internal/shedrecipe/entries_planwrite.go:15`.
2. MEDIUM, consistency. D5 offers `goto Webster` "to run without regression". With a failed or missing capture under scope `partial` or `full`, D6 then halts Stuck at `Drift-Verify` because there is no passed capture record. `Drift-Await` is also Stuck again on every Webster reroute through the plan perch, since `awaited_at` is set only on a passed await. The chosen route halts twice or more instead of running without regression. Evidence: `decision-record.md:116-120`, `:127`; `loom-recipe.yaml` Webster `on_reroute: Plan-Burler-Review`.
3. MEDIUM, design. D5 says `awaited_at` lets the run pass "after ... an adopt on another machine". But the capture record, baseline directory and skip file live under never-tracked `.lyx/drift/` (Constraints). After an adopt, `Drift-Verify` on the new machine finds no passed capture record and halts. A capture still running at the adopt is ended by `EndOwnedElsewhere`. The record is silent on both outcomes. Evidence: `decision-record.md:120`, `:161`, `:127`; `internal/tendercli/deps.go` `EndOwnedElsewhere`.
4. MEDIUM, design. Under scope `full`, the halt names "`lyx drift accept` and resume" as a way forward, but the accepted list persists in the scope record. The acceptance criterion "any acceptance ends `Drift-Verify` Stuck" therefore re-halts on every resume. No named way forward (`goto Describe` is not offered here) reaches Done, against `PATTERN-refusal-way-forward`. The `full`-scope halts are also unclear on whether resume respawns an agent. Evidence: `decision-record.md:142-143`, `:151`, `:194`.
5. LOW, design. `verify_max_runs` counts "per agent session" in a Go-written counter, but the record never states when the counter resets: on respawn, on resume after a halt, or never. Resuming a past-budget halt either gives the new agent no runs or an unbounded series of sessions. Evidence: `decision-record.md:42`, `:135`.

**Tool calls**

- One `grep` of `internal/shedrecipe/entries_planwrite.go` for "commit". My context lacked the line confirming that the `PlanWrite` registry row carries the plan-directory rotator as its fresh-spawn preparation.
