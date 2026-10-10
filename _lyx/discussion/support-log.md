# Support log: quick-recipe

## Interview

The session ran autonomously, so every turn below is the writer's own question and the answer it reached from the board, the code and the repo rules.

1. Does the task exist and what does it fold in?
   The board entry `quick-recipe` holds the darn recipe plus four fixes: per-attempt verify logs (#502), Finalize blocked by the parent weft's batten records (#501), teardown leaving a board claim, and Friction-Reflect misattributing a dirty tree.
   Related entries read: `driver-context` (done, the Go step loop the driver stencil now assumes), `orch-owns-the-loop`, `shedrecipe-capabilities`, `run-message-routing`, `verify-watchdog`, `lyx-cleanup`, `lyx-halt`, `master-self-fix`, `convergence-plan`.
   None conflicts; `verify-watchdog` and `lyx-cleanup` stay out.
2. How is a recipe chosen today?
   The board entry's `recipe` field, read fresh by batten's `Seed-Child`, validated against `shedrun.RecipeNames` and refused unless `loom`, because `Run-Shed` spawns `lyx loom start` unconditionally.
   `internal/shedcli` maps recipe names to arming functions.
   Answer: add `darn` to the vocabulary, admit it in batten, map it to loom's arming.
3. New CLI module or loom's?
   Loom's verbs are all status-file driven and the orch's procedures are written against them.
   Answer: no `lyx darn`; D1.
4. How does a failed verify return to the same producer?
   The gate mechanism re-prompts the live session up to `attempts` times with the findings; `Webster-Burler` already carries a `verify` gate and `webster.yaml` has a config-sourced attempts count as precedent.
   Answer: a `verify` gate on the `Darn` row with its attempts stamped from `darn.yaml`; D4.
5. Where does the verify command come from without a plan?
   Publish and Finalize take `VerifyCommand` as a told closure; the gate parses the plan.
   Answer: tell the gate too; darn's source is `darn.yaml`'s `verify`; D6, D7.
6. Where should the PR description come from?
   Describe reads the decision record and webster's summaries, none of which darn has.
   Answer: the darn writer writes it, gated by the existing `description` gate; D5.
7. How does a PR-Gate rejection reach the darn producer?
   PR-Gate's Stuck bounces to the row producing the diff; the rejection record holds the findings.
   Answer: `on_stuck: Darn` in a shared segment and a rejection token in the stencil; D3, D4.
8. Why does Finalize see the parent weft's dirt?
   `Merge`'s pre-merge sync fast-forwards both checkouts of the parent pair and refuses a dirty one; batten commits its status only at transitions and skips while the parent pair is mid-merge.
   Answer: Finalize commits the parent's `_lyx/shed/` subtree first; D9.
9. Where does an abandoned run's claim get cleared?
   Every pair teardown goes through `internal/pairteardown`.
   Answer: the composite settles it, marking a landed run done and clearing only an abandoned run's claim, judged by batten's state for the slug; D10.
10. How should Friction-Reflect learn the parent's status?
    Go writes the halt note; fabric owns git.
    Answer: a Go-written section in the halt note; D11.
11. How are verify logs kept per attempt?
    `verifytree` writes one fixed `verify.log`; Publish copies it into the failure record.
    Answer: one numbered log per run, bounded at twenty; D8.
12. Which skills, color and model for the darn writer?
    No existing implementer role fits exactly; the webster forks run on the master's model.
    Answer: `scribe:prose`, `scribe:code-quality`, a new `darn` color segment, `opus[medium]`; D4, D7.

## Rejected alternatives

- A `lyx darn` command tree.
  Rejected: it would duplicate `start`, `approve`, `reject`, `resume`, `goto` and the driver launch, and force the orch's run loop to branch on recipe.
- A `--recipe` flag on `lyx loom start` for hand-started darn runs.
  Rejected as unasked: the operator picks the recipe at task creation and batten is the path that keeps the prime clean.
- A `Describe` row in darn with placeholder inputs.
  Rejected: its stencil reads artifacts darn does not produce, and a second session adds cost for a one-author change.
- A darn producer that writes a tiny plan or record file carrying `## verify:`.
  Rejected: it would add a parsed artifact format under `PATTERN-sole-parsers` to express one configuration line.
- Per-worktree `darn.yaml`.
  Rejected: the verify command runs any shell command in the task worktree, and a task could weaken its own gate through its tree; landing's `publish_verify` already states the rule.
- Impacted-set narrowing in the darn gate, with Publish running the full verify.
  Rejected: a Publish failure escalates to a human, so the budgeted gate must run what Publish runs or the budget protects nothing.
- Making the merge guard ignore `_lyx/shed/` dirt on the parent weft.
  Rejected: the pre-merge fast-forward would still refuse when upstream touched those paths, and ignoring dirt weakens a guard where committing it keeps the guard strict.
- Having batten commit its status on every write.
  Rejected: the no-op-transition skip exists to avoid one commit per bounce, and the skip while mid-merge would still leave dirt for a sibling landing.
- Clearing the claim in batten's `Worktree-Teardown` row and in `lyx fabric remove` separately.
  Rejected: two call sites for one rule; the teardown composite is already the chokepoint.
- Clearing the claim whenever the entry is not done, judging the board alone.
  Rejected: it clears a landed run whose done mark failed and releases a run batten still holds.
- Setting a word such as `stopped` as the released status.
  Rejected: an empty status is what the entry had before the run, and batten's notices already tell the orch why the run ended.
- The reflection agent running `lyx fabric status` in the parent.
  Rejected: a deterministic read belongs in Go, and the halt note is the place Go already writes.
- Appending attempts into one `verify.log` with headers.
  Rejected: a per-run file is simpler for a reader and lets the failure record point at a file nothing rewrites.
- A stencil sentence telling a re-entered `Darn` spawn to inspect the task branch for its own earlier commits.
  Rejected: Go already holds the run's history and the Publish failure record, so a rendered prior-work token states why the spawn re-entered without the agent reconstructing it.
- Unbounded verify-log retention.
  Rejected: a long-lived prime's `.lyx/verify/` would grow without end.

## Review rounds

_No rounds yet._

## Question ledger

| # | Question | Status | Resolution |
|---|---|---|---|
| 1 | CLI module for darn runs | resolved, auto-pick | `lyx loom` serves darn; D1 |
| 2 | How batten admits a darn child | resolved | admit list gains `darn`; D2 |
| 3 | Darn recipe rows and routing | resolved, auto-pick | D3 |
| 4 | Retry mechanism and budget source | resolved | verify gate, attempts from `darn.yaml`; D4 |
| 5 | Who writes the PR description | resolved, auto-pick | the darn writer; D5 |
| 6 | Verify command source for darn | resolved, auto-pick | `darn.yaml` `verify`, told to gate and landing; D6 |
| 7 | Config module scope | resolved, auto-pick | hub-wide `darn.yaml`; D7 |
| 8 | Default writer model and timeout | resolved, auto-pick | `opus[medium]`, 180 min; D7 |
| 9 | Verify log naming and retention | resolved, auto-pick | `verify-<n>.log`, newest twenty; D8 |
| 10 | Finalize and the parent weft's records | resolved, auto-pick | commit `_lyx/shed/` first; D9 |
| 11 | Where the claim is cleared and to what | resolved, auto-pick | pairteardown; landed marks done, abandoned clears to empty, a `running` batten run keeps it; D10 |
| 12 | How Friction-Reflect reads the parent | resolved, auto-pick | Go-written halt note section; D11 |
| 13 | Skills and color for the darn writer | resolved, auto-pick | `scribe:prose`, `scribe:code-quality`, `darn` segment; D4 |
| 14 | Whether the darn gate narrows by impacted set | resolved, auto-pick | no, full verify; D6 |
| 15 | Whether `lyx loom start` needs a recipe flag | resolved, auto-pick | no; batten is the start path; Open risks |

No environment fault was met; every `lyx board` read answered and the worktree was read-only for this session.
