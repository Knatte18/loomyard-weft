```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens, which checks each pair of decisions that act on the same state against each other and then checks the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
1. D5 (Stuck names `goto Webster` "to run without regression") × D6 (a scope with no passed capture is Stuck, naming `goto Describe`): broken.
2. D5 (HEAD moved names only `goto Webster`) × D6 (a passed capture record exists, so the compare runs): broken in wording. Regression does run, which contradicts "without regression".
3. D6 (under `full`, "a session that accepted any drift is Stuck") × D6 (the halt's way forward is "`lyx drift accept` and resume") × D7 (the accepted list does not record who accepted) × the AC "any acceptance ends `Drift-Verify` Stuck": broken.
4. D2 (the arbiter may settle with `lyx drift accept`) × D6 (`full` is Stuck "so the parent rules"): broken. It folds into pair 3.
5. D2 (loom's `Drift-Baseline` `on_done` is `Plan-Write`) × D5 and D3 (`goto Drift-Baseline` as the way to capture again): broken.
6. D4 (on Windows a timed-out capture keeps the lock until its command exits) × D3 (re-entry captures only when no one holds the lock) × D5 (Stuck after timeout plus one minute, naming `goto Drift-Baseline`) × the AC "goto Drift-Baseline after a failed or dead capture captures again": broken. This applies to Windows only.
7. D6 (`verify_max_runs` counts per agent session) × D6 (halt and resume re-enters the row): the bound is unstated, so the budget's reset is undefined.
8. D1 (with the flag on, undeclared commands are refused) × D2 (ungated `Drift-Await`): held, since `Drift-Await` runs no command.
9. D2 (flag turned off mid-run) × D5 and D6 (`Drift-Await` still holds, `Drift-Verify` is Skipped): held.
10. AC1 × D2 (darn has no `Drift-Await`): broken in wording.
11. D3 (`lyx drift scope` only at `Drift-Baseline`) × D6 (an amendment never re-scopes): held.
12. D3 (a present scope record skips the agent) × D5 (`awaited_at` in the scope record) × D4 (a fresh capture writes a fresh record): held.
13. D4 (the main capture refuses a dirty tree and anything off the branch point) × D6 (the side capture needs neither): held.
14. D4 (the row that next ends commits the log) × D5 Stuck paths: held, since Stuck is an end.
15. D6 handoff order (dirty tree, then the gate, then the compare last) × the 3-attempt re-prompt: held.
16. Constraint "no row builds between `Drift-Baseline` and `Drift-Await`" × D3 detached capture beside Plan: held.
17. D7 (accept only at the `Drift-Verify` row) × D6 (agent-only verbs): held.
18. AC "HEAD moved before the first await halts" × D5 `awaited_at`: held.

**Findings**

- **BLOCKING · consistency · D6, D7, Acceptance criteria.** Under scope `full`, the halt's own way forward cannot clear it.
  - D6 has a `full` session that accepted drift end Stuck "so the parent rules", and names "`lyx drift accept` and resume" as the way forward.
  - The AC says "Under scope `full`, any acceptance ends `Drift-Verify` Stuck".
  - D7's accepted list records no accepter.
  - So if the rule reads the accepted list (the AC wording), the parent's or arbiter's accept-and-resume halts again, with no way out but `goto Describe`.
  - If it counts only the agent session's acceptances (the D6 wording), the arbiter, which D2 lets accept, settles a `full` drift before the parent ever sees it.
  - Evidence: `decision-record.md:56`, `:142-143`, `:151-152`, `:194`.
- **MEDIUM · consistency · D5 × D6.** `goto Webster` "to run without regression" leads to a second halt.
  - After a failed or dead capture, `Drift-Verify` finds no passed capture record and is Stuck, naming `goto Describe`.
  - So the operator's earlier choice costs a second halt that the record never mentions.
  - After a HEAD-moved halt the passed capture still exists, so `Drift-Verify` does compare, and "without regression" is false there.
  - Evidence: `decision-record.md:118-119`, `:127`.
- **MEDIUM · design · D2 × D5 (D3).** In loom, the way forward D5 names, `lyx loom goto Drift-Baseline`, re-runs `Plan-Write` and the whole Plan perch.
  - `Drift-Baseline`'s `on_done` is `Plan-Write`, and `Plan-Write`'s fresh-spawn preparation archives the plan directory.
  - So recapturing silently throws away the approved plan and re-reviews it.
  - The record neither states this cost nor gives a recapture route that returns to `Drift-Await`.
  - Evidence: `decision-record.md:53`, `:118`; `internal/loomshed/planwrite.go:96-100`; `contracts/recipes/loom-recipe.yaml` (Plan rows follow `Discussion-Bouncer`).
- **LOW · design · D4 × D3 × D5 (Windows).** A timed-out capture on Windows is recorded `failed` but keeps the lock.
  - `Drift-Await` is Stuck naming `goto Drift-Baseline`.
  - D3 starts no capture while the lock is held, and leaves the row's outcome in that case unstated.
  - So the AC "goto Drift-Baseline after a failed … capture captures again" does not hold until the command exits.
  - Evidence: `decision-record.md:83`, `:102`, `:114-115`, `:188`.
- **LOW · design · D6.** The record never says when the `verify_max_runs` counter resets ("one agent session").
  - A halt and resume that spawns a new agent either inherits an exhausted budget, so both verbs refuse at once, or gets a fresh one, so the budget is unbounded across resumes.
  - Evidence: `decision-record.md:42`, `:135`, `:142`.
- **NIT · consistency · Acceptance criteria, first bullet.** "a loom and a darn run record … `Drift-Await` as Done", but darn has no `Drift-Await` row (D2).
  - Evidence: `decision-record.md:54`, `:182`.

Tool calls: none; my inherited context held everything this lens needed.
