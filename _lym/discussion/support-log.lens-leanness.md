```yaml
lens: leanness
outcome: reported
```

Leanness lens: I looked for content in the decision record that should be moved or cut.

1. **MEDIUM — D7 and D8 (outcome `failed`).** The `failed` outcome can never be set, so its machinery is dead content and should be cut or redesigned.
   - Evidence: D3 puts the leader's own entry first in every batch, and D7 halves to "the chain's first half". So every one-entry prefix is the leader's own entry.
   - Walk it through with culprit B in [L, B, C, D]. The full batch fails, then [L, B] fails, then [L] passes and L lands alone. B, C and D stay queued with no outcome. B leads next, and [B] is again the leader's own entry, so B goes to its fixer.
   - The case "A failing prefix of one that is another run's entry" is unreachable, so the outcome `failed`, its tree, "drops that entry … batch of the rest is built again", the D8 `failed` branch and the D7 acceptance bullet are all moot.
   - The design still reaches the culprit's fixer, but only by landing the leader alone and making the culprit lead later. The record should state that path, or cut the dead branch. This overlaps the walkthrough lens; I raise it here because the content is removable as written.
2. **MEDIUM — Open risks, first bullet (leader's gate override applies to the whole batch).**
   - It is a bound of D5 that the writer settled, not a question for the parent. Step 4 limits Open risks to parent-level questions: an outdated entry, a conflict with another run, doubt about the task's purpose.
   - Move it into D5 as its stated bound. The rubric's attack-surface rule wants the bound inside the decision.
   - The second bullet (an interaction failure blames the later run) is also a bound of D7. It belongs in D7 too.
3. **LOW — Auto-mode assumptions.**
   - All three bullets repeat D1, D9 and Scope "Out" word for word.
   - The section is required, but it could name the auto-picked decisions by id instead of restating them.
4. **LOW — Scope "Out" and Auto-mode assumptions and support log, "gating prefixes side by side".**
   - The exclusion is stated three times. Its reasoning belongs in Rejected alternatives, where it already is.
   - Keep the one Scope "Out" bullet and drop the auto-mode repeat.
5. **LOW — Scope "Out", "Pull-request landings … (D10)".** This repeats D10 inside the record. The cross-reference is enough on its own, or the bullet can be dropped.
6. **LOW — Constraints, `beam.pull_request` is `off` … "so most landings here are local and can batch".**
   - This is motivation, not a constraint the plan writer acts on. It belongs in the support log, where Turn 1 already records it.
7. **LOW — Constraints, `PATTERN-landing` bullet.** It restates D12. Drop it, or reduce it to "D12 changes it".
8. **LOW — Notes for the plan writer, second bullet (`gitrepo.Repo.MergeTree` takes commits …).** It repeats D4's mechanism, since each step writes its commit before the next merge. Cut it.
9. **LOW — Goal.**
   - The second and third sentences spell out mechanism that D3 to D7 own: the leader, `landing.batch_max`, one fast-forward, halving.
   - The Goal could keep only the aim (one gate per batch, gated exactly as landed) and the culprit's return to its fixer.
10. **LOW — Acceptance criteria, D6/D8 bullet, "Tier 1, `internal/landingshed` or `internal/loomcli`".** The "or" leaves the test's home to infer. Name one package. This is outside my lens.

Further tool calls: none.
