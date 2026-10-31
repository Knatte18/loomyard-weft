```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism on one concrete case and checked the end state against the decision's purpose.

**Decisions walked:**
- **D1:** sampler run with `code_review` false, from Preflight to Friction-Reflect along `on_done`. Purpose met, but the acceptance criterion fails (finding 1).
- **D2:** a task naming `light`, then a task naming no profile. Seed-Child writes the block's `sampler.code_review: true` and the loom values to the local layer; with no profile and no `defaults.sampler`, the run uses the hub config with the perch off. Purpose met.
- **D3:** `code_review` false, hub `code_max_bounces` 2, Finalize fixer over the caps. Once `rerouteReviewed` reads the gate, it reads false and Finalize halts. Purpose met; one named reading has no effect (finding 4).
- **D4:** agent strand runs `decision add --by operator` at Sampler-Session after a resume. Today `decisionVerb` refuses the named strand (decision.go:648-654). Widened, the re-recorded strand matches the exported name; the notice skips the caller. Purpose met.
- **D5:** first entry, then the strand dies and the run resumes. `ensureSession` starts, records, then resumes by session id and re-records. Purpose met.
- **D6:** verb run from the operator's shell while the run is halted at the row because the strand died, then `lym loom resume`. The purpose is not met when the session cannot be resumed (finding 2). A separate case, a stale flag across a Finalize reroute, cannot happen (finding 3).
- **D7:** child at Sampler-Session with the operator silent past `step_idle_timeout_min`. A list-based activity reading reports activity now and dobby's notice is quiet. Purpose met.
- **D8:** the agent writes a board result. It goes through `lym board` as a note, and the constraint fact allows notes. Purpose met.
- **D9:** a tmux trial. A `tmux -L &lt;slug-name&gt;` server outside lym, killed before `sampler-done`. Purpose met, bounded by the stencil alone.

**Findings:**

1. **MEDIUM, consistency.** Acceptance criteria, bullet "D1, D2, D3: with `code_review` false the three Code-Review rows are recorded Skipped".
   Only Code-Bouncer can be recorded Skipped. A gate that does not hold routes by the row's `on_done`, and Code-Bouncer's `on_done` is Describe. Code-Burler-Review and Code-Burler-Fix are reached only through `on_stuck` edges, so they are never entered and get no Skipped entry.
   Evidence: `internal/shedengine/entry.go:51-74` (skip routes by `def.OnDone`), `contracts/recipes/darn-recipe.yaml` (Code-Bouncer `on_done: Describe`; the Burler rows sit on `on_stuck`).

2. **MEDIUM, design.** D6 Bound ("accepted while the run is halted at the row, so the next `lym loom resume` ends the row Done at once") against D5's session start on entry.
   The record never states whether entering the row reads the done flag before or after it ensures a session. If the session is ensured first, as PR-Review's `ensureSession` does on each call, then a run halted because its strand died cannot end Done when the resume of its recorded session fails non-transiently. It is Stuck again with a Halt naming `lym loom resume`, and every later resume repeats that, so the promised Done never happens. Even when the resume succeeds, the session is brought back only to be stopped. The record does not say what happens to a flag written while the run is halted at a row whose session cannot be resumed.
   Evidence: `internal/landingshed/prsession.go:63-87` (`ensureSession` resumes, and a non-transient failure is a `sessionStartHalt`); decision-record D5 last bullet, D6 Bound.

3. **LOW, consistency.** D6's first bullet ("a return to the row after it ended Done, by Finalize's reroute or a `goto`, raises it") and the matching D6 acceptance bullet ("a done flag written before a reroute or `goto` back to the row").
   D1 keeps `on_reroute: Code-Bouncer`, so a Finalize reroute never returns to Sampler-Session. Only a `goto` can. The reroute case describes a path that does not exist, and its test cannot be built.
   Evidence: `contracts/recipes/darn-recipe.yaml` (Finalize `on_reroute: Code-Bouncer`); decision-record D1.

4. **LOW, consistency.** D3 ("the landing's `CodeReviewOff` in `internal/loomcli/landingdeps.go` read a perch whose rows the gate leaves out as off").
   `CodeReviewOff`'s only consumer is Parent-Align, and D1 gives sampler no Parent-Align row. Changing this reading has no effect on a sampler run; only `rerouteReviewed` decides whether Finalize reroutes or halts.
   Evidence: `internal/landingshed/parentalign.go:435`, `internal/landingshed/deps.go:320-322`, `internal/loomcli/landingdeps.go:272`.

**Tool calls:**
- `grep` of `internal/shedengine` for how `goto` records leaving a row (`goto.go:135`, OutcomeGoto). My context had the Done-count stamp but not what history a `goto` writes, which decides whether the stamp moves.
