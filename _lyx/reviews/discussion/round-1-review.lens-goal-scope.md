```yaml
lens: goal-scope
outcome: reported
```

Task: goal-scope lens. Check the decision record against its board entry and any `Decision for regression-baseline-verify:` notes.

**Goal points of the entry, and whether the record meets each**

- Regression is no test tier and has two commands, capture and compare; the result is a drift judgement for an agent. **Met** (Goal, D1).
- Regression has its own config block in the module owning the Drift rows: the flag, capture, compare, skip passing and models. **Met** (D1).
- "Its commands still take a gate slot, so it never runs beside other heavy test runs." **Not met in purpose.** D4 takes one slot "within the hub's slot count", so a capture runs beside other tier runs (finding 2).
- Discussion knows nothing of regression and its stencil is unchanged. **Met** (Scope Out).
- `Drift-Baseline` after Discussion: a Sonnet agent scopes off, partial or full, writes the expected drift and starts the capture in the background in the task worktree. **Met** (D2, D3, D4).
- `Drift-Await` is mechanical and halts on a failed or timed-out capture, with no later fallback. **Met** (D5).
- `Drift-Verify` "after Webster and the Code perch, before Describe and Parent-Align". **Cannot be met as written.** Since `fc75aef79` landed, the two halves contradict each other. The record picks one reading and parks the question under Open risks (finding 1).
- `Drift-Verify` does mechanical compare with no LLM on a pass, and spawns one strong agent within a budget that judges and fixes, writes its reasoning and is no perch. **Met** (D6).
- `Drift-Verify` "escalates to the orch when unsure or past its budget". **Partly met.** D2 lets the arbiter settle the halt with `lyx drift accept` before the orch sees it (finding 3).
- Darn: `Drift-Baseline` before Darn-Write, reading the board entry, with a foreground capture and no `Drift-Await`; `Drift-Verify` after Darn-Write, before Code-Review. **Met** (D2, D3).
- Operator decision: no existing producer's behaviour changes. **Met** (Scope Out).
- Operator decision: one clearly named key, off by default, and this hub stays off. **Met** (D1, Scope Out).
- Operator decision: with the flag off, the rows are "skipped like the PR rows". **Departed from for `Drift-Await`** (D2), which records Done, not Skipped (finding 4).
- Operator decision: `Drift-Verify`'s fixes need no review, and if one is added it is a solo agent. **Met** (D6).
- Operator decision: no drift report format is pinned in `contracts/specs/`. **Met** (Scope Out).
- Operator decision: Models lives on OSL-1046, and everything is built and tested here without Models. **Met** (Scope Out, last acceptance criterion).
- Models harness: the baseline is never overwritten, and a refresh backs up first. **Met** (D4 backup; the only refresh is before the first code change).
- Models harness: the `*.cs` diff trust check. **Replaced** by the stricter branch-point check, with the rejection recorded in the support log. No finding.
- Brief: "switchable per task for expected drift". **Met** (D3 scope).

**Decision notes**

`lyx board list` names no `Decision for regression-baseline-verify:` note, so there was none to read and none I failed to reach.
The record has no `Added after Discussion` entries.

**Findings**

1. **MEDIUM · decision.**
   Location: Open risks bullet 1; D2.
   Summary: The board entry places `Drift-Verify` "after Webster and the Code perch, before Describe and Parent-Align". Landed commit `fc75aef79` makes that impossible, because `Parent-Align` now runs before the Code perch.
   The record chose "after the Code perch" and left the choice in Open risks, so nothing routes it to the parent.
   Question for the parent: should `Drift-Verify` sit after `Code-Bouncer`, comparing a tree that carries merged parent commits and accepting drift traced to them? Or should it sit between Webster and `Parent-Align`, leaving the Code perch's fixes uncompared?
   Evidence: board entry body, "## Design (operator, 2026-10-24)" item 3; `contracts/recipes/loom-recipe.yaml:357` (Parent-Align, `on_done: Code-Bouncer`); `decision-record.md:175-176`.

2. **MEDIUM · design.**
   Location: D4 ("holds one `internal/gateslot` slot … so it runs within the hub's slot count").
   Summary: The entry says regression's commands take a gate slot "so it never runs beside other heavy test runs". With one slot in a multi-slot hub, a capture or compare runs beside tier runs, so D4 drops the entry's stated purpose.
   The writer can settle it by holding every slot through the multi-slot acquire (drain lock), or by stating the departure as a bound.
   The support log records the weakening as a lens fix and not as a departure from the entry.
   Evidence: board entry body, "Its commands still take a gate slot, so it never runs beside other heavy test runs"; `decision-record.md:95`; `support-log.md:38`; `internal/gateslot/doc.go:15-17` (multi-slot acquire exists).

3. **MEDIUM · design.**
   Location: D2 ("The arbiter may settle a Drift halt with the way forward the halt names, `lyx drift accept` included"); D6 and D7.
   Summary: The entry escalates the verify agent's "unsure or past its budget" case to the orch. D2 instead lets the cold arbiter agent accept drift that the strong agent declined to judge.
   D2 does not state what such an acceptance lets through, or whether it is bounded below the parent.
   Evidence: board entry body, Design item 3, "escalates to the orch when unsure or past its budget"; `decision-record.md:56`; `internal/shedarbiter/doc.go:4`.

4. **LOW · decision.**
   Location: D2, paragraph 2.
   Summary: The operator decision of 2026-10-25 says that with the flag off, the Drift rows "are skipped like the PR rows are today". D2 departs for `Drift-Await`: the row stays ungated and shows Done.
   The departure overrides a dated operator decision, and complying would reopen the mid-capture hazard D2 names, so the fixer cannot settle it.
   Question for the parent: does the operator accept `Drift-Await` showing Done rather than Skipped with the flag off, to keep Webster held behind a capture already running?
   Evidence: board entry body, "Operator decisions (2026-10-25)" bullet 3; `decision-record.md:58-60`; `support-log.md:62`.

**Tool calls**

- `lyx board list`, filtered for this task and its decision notes. The context held only `lyx board get` for this task, not the list my lens requires.
