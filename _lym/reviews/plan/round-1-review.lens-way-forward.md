```yaml
lens: way-forward
outcome: reported
```

Task: a way-forward review of each refusal, stop or recovery step the plan's cards add.

**Per refusal: the next step it names, and whether it progresses**

- **Card 2, loom or darn fixer `escalate` halt at Finalize or Parent-Align (`EscalateStopPrefix`).** Next step: "record the deciding decision with `lym loom decision add` in the task worktree, then `lym loom resume`". Progresses.
  - The escalate gives its turn back: `fixerrun.go:257-259` decrements `Spent` and writes the record. So the resumed row has attempts left, and on a still-failing gate it re-spawns the fixer, which reads the decision record afresh.
  - The arbiter's own route, `decision add` plus a `resume` with the halted row as goto, reaches the same state.
  - Darn has a decision record (`internal/loomcli/decision.go:773`), so `decision add` is not refused there.
- **Card 2, pawl fixer `escalate` halt (`LandingStopPrefix`).** Next step: "settle the conflict on the task branch, then `lym loom resume`; a settle past the fixer's caps halts over-cap, and the resume after that halt accepts the commits". Progresses at both rows. The second clause is only accurate at Finalize (finding 1).
  - At Finalize, `chargeTurn` (`fixerrun.go:212-215`) persists `Baseline`. The resume's passing gate then runs `judgeFixerChange` from that `Baseline`. With `RerouteReviewed` nil on pawl, an over-cap settle halts over-cap, and the next resume takes `overCapResume` and lands.
- **Card 2, batten re-step skip of an escalate halt (`restepSkips`).** Next step: the arbiter through a Go driver or the eligible-stop hold, else the orch. Progresses. Batten's eligible-stop hold ends at the hand-on or at a bounded backstop, then notices the orch (`internal/battenshed/doc.go:45-47,58`), so a driverless child is not held forever.
- **Card 3, arbiter-first routing of an escalate stop.** Next step: the arbiter's `resume` re-runs the row; a hand-on, an arbiter that is off and a spent budget hand back to the orch, whose step is the card 2 way forward. Progresses.
- **Card 4, `restep_hold` at a stop for the LLM driver.** Next step: the arbiter, then the orch's `lym loom resume`. Progresses. The driver stencil sends a `blocked` stop to `## Arbiter` first (`shed-template-driver.md:60`), and `restep_hold` withholds only the build re-launch and the binary watch.
- **Card 1, a large merge or conflict going to the `finalize` gate.** It adds no new refusal. The spent and over-cap halt or reroute that remains keeps the `keepFixerCommits` way forward ("review and fix on the task branch, then `lym loom resume`"). Progresses.

**Findings**

1. **LOW, consistency, card 2 (`02-escalate-halt-prefix.md`, Intent, the pawl way forward of `fixerOutcome.haltReason`).** The clause "a settle past the fixer's caps halts over-cap" is true at Finalize but not at a pawl Parent-Align with a Code-Review perch budget above 0.
   - There the operator's settle goes to the Code-Perch unjudged and never halts over-cap.
   - The halt still progresses; only its text is wrong for that row.
   - Evidence:
     - `internal/landingshed/doc.go:113`: with a budget above 0 the change goes to the Code-Perch unjudged.
     - `contracts/recipes/pawl-recipe.yaml:57-59`: pawl's Parent-Align `on_done` is `Code-Bouncer`.
     - `internal/landingshed/parentalign.go:421-424`: `judgeFixerChange` judges only when no review follows.
   - Fix: scope the clause to Finalize, or drop it. Decision D3's pawl way forward has only "settle the conflict on the task branch, then `lym loom resume`".

**Tool calls**

1. `internal/landingshed/fixerrun.go` grep for record writes: the manifest held only `haltReason`, not whether escalate persists `Spent` and `Baseline`.
2. Driver stencil arbiter lines and pawl recipe rows: the manifest held only stencil lines 97–144, not the arbiter-first rule, and not pawl's row order.
3. `internal/loomcli/decision.go` recipe refusal and `fixerrun.go:205-262`: to check whether `decision add` refuses darn, and how the escalate turn is given back.
4. `internal/battenshed/doc.go` eligible-stop hold: to check that a stop batten's re-step skips still reaches the orch.
