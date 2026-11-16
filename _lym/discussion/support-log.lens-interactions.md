```yaml
lens: interactions
outcome: reported
```

Task: check every pair of decisions in the Weaver record that act on shared state for contradictions, then check each acceptance criterion and constraint against the decisions it touches.

**Pairs checked:**
- D5 × D7, the ladder's attempt bound against the extra re-entry attempt: broken.
- D7 × D11, the re-entry check against the post-batch checks: broken.
- D7 × D6, re-entry after a `NEEDS_PARENT` stop: broken by the same gap as D7 × D11.
- D7 × D10, re-entry after a verify stop: broken; the case is undefined.
- D4 × D12, a gated strand against attach-on-resume: broken; the order is ambiguous.
- D10 × the row's `gates` entry, two attempt bounds: broken; the bound is unstated.
- D2 × D1 and D6, the "Stuck only when" list against the reroute: broken in wording.
- D2 × D10, the Stuck reason's contents for a verify stop: broken in wording.
- D4 × Constraints (PATTERN-reed-panes), "non-interactive" against "interactive": broken in wording.
- D9 × implementer-spec, a commit per batch against "commits once at Done": held. The spec text lags, so it is a LOW finding.
- D8 × D9, journal entries written per attempt but committed per batch: held. `_lym/weaver/` sits on the weft side, so the warp tree's clean check is not affected.
- D3 × D11, re-partitioning after renames rewrite pending cards: held.
- D3 × D12, a resume uses the batch recorded in state and does not re-partition: held.
- D3 × D6, the plan-conflict reroute and done cards after the plan is revised: held if done cards are matched by slug. The record doesn't say, so it is a LOW finding.
- D5 × D10, the fixer climbing the ladder: held.
- D8 × D2, the arbiter reaching the journal through the Stuck reason: held.
- D1 × D10, Done meaning the verify passed: held.
- The acceptance criteria hold against D1–D12, except the D7 criterion, which inherits the D7 × D11 gap.

**Findings:**

1. MEDIUM — D5 × D7. D5's bound says "a batch gets at most one attempt per rung, so the ladder's length caps its attempts before the stop chain". D7 adds one more last-rung attempt on each re-entry, so a batch can run many times on the last rung. The bound is false as written. Evidence: D5's Bound line, and D7's "one more attempt on the ladder's last rung". Fix: D5's bound should count the re-entry attempt, which the arbiter spawn cap bounds.

2. MEDIUM — D7 × D11. On re-entry, D7 records a stopped batch done when its card gates pass at HEAD and the tree is clean. That skips the post-batch checks D11 shares: done checks, handle binding, the scope guard and the rename rewrite of pending cards. A card gate can also pass on a package the batch never changed. A batch could then be recorded done with its cards undone, and a rename made during the arbiter's fix would never reach the pending cards. Evidence: D7's first two sentences; D11 lists the post-batch checks every batch end runs. Fix: run the full post-batch checks on re-entry.

3. MEDIUM — D7 × D10. D7 only covers "a batch in state marked stopped". After a stop because the verify fixers ran out, no batch is stopped, so re-entry is undefined. It could run the verify again, run one last-rung fixer, or loop over batches with nothing left. Evidence: D10's "A failure after the last rung stops the row"; D7's wording.

4. MEDIUM — D4 × D12. D4 gives each attempt's strand the card gates as a must-pass gate. Under PATTERN-shuttle-provider-seam a gated strand has no done-when list, so reed relaunches a finished gated strand after a crash and types no prompt. D12 lists attaching to a live strand before recording a finished report. A resume could then attach to that idle relaunched strand and wait until it times out, although the report already exists. Evidence: PATTERN-shuttle-provider-seam, "The done-when list"; D12's order. Fix: check for an unrecorded finished report before attaching.

5. LOW — D10. The row's drift entry carries `attempts: 3`, a shuttle re-prompt count for Merriam. D10 sends any gate failure to fixers on the ladder, so two attempt bounds apply and the record doesn't say which one wins. D10 also doesn't say whether a fixer's commit after a drift failure runs the verify again before the drift check.

6. LOW — D2 against D1 and D6. D2 says the row "returns Stuck only when" four causes occur. Its list omits the plan-conflict Stuck that takes the reroute, which D1 and D6 both name. D2's Stuck reason also "names the batch, its cards", which doesn't fit a verify stop; it should name the findings.

7. LOW — D4 against Constraints. D4 says each attempt starts a "non-interactive shuttle strand". The PATTERN-reed-panes constraint says "every batch and fixer strand is an interactive reed session". Shuttle uses "non-interactive" to mean no operator, but the record reads as contradicting itself. One term should be used.

8. LOW — D9 against `contracts/specs/implementer-spec.md`. The spec says Go commits the run directory "once the row reports Done", but D9 commits after every batch. The spec's Out section needs the per-batch commit, or a note that an implementer may also commit earlier.

9. LOW — D3 × D6 reroute. After Plan-Burler-Review revises the plan and the row runs again, D3 "drops the cards already done". The record doesn't say done cards are matched by slug, so a renumbered plan could drop the wrong cards.

Further tool calls: none; my context held everything this lens needed.
