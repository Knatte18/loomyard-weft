# Support log — rework-generations

## Interview

Autonomous session: no operator answered.
Each turn below is a question the design had to settle, with the self-picked answer; the board entry for `rework-generations` was the only operator input.

1. What does "archived (git keeps them)" mean on disk?
   Explored `NewPlanDirRotator` (Plan-Write's `archive-*/` rotation), `websterengine/archive.go`, and `prrework.go`.
   The rework session needs the retired cards readable to write new ones, and the task worktree cannot read the plan at HEAD (the `_lyx` junction is excluded from git there), so deletion alone is not enough.
   Picked: move the retired generation into the rework round's own directory, `round-<N>/prior-generation/`, and commit the move.
2. What belongs to a generation besides the cards?
   Webster resumes the same run after rework (`RebaselinePlanFingerprint`), so its state carries every earlier batch.
   The Plan-Review and Webster-Review run directories also accumulate rounds; a judge's ledger carry-forward and a Burler's prior-round list would point at cards that no longer exist.
   Picked: plan directory, Webster's durable run record, and both review run directories all move.
3. Card numbering in a new generation: restart at 1 or continue?
   The plan spec numbers cards `1..N` and pins the `N: ` commit-subject prefix as the git-log resume trail; coverage maps and findings cite card numbers.
   Picked: continue, via an optional `first_card` overview key.
4. How does Plan-Review get conditional routing when shed rows have one Done edge?
   Read `internal/shedengine` routing docs: OnDone and OnStuck only, no conditional edge.
   Picked: route every rework generation to Plan-Bouncer and give that row a skip seam for exempt generations, so approval stays on the one row that owns the approve seam.
5. Who decides "changes a classifier, verb or guard"?
   Picked: Go, mechanically, from card type labels: all-`Prosa` is exempt, anything else is reviewed.
6. What does Plan-Review measure a rework generation against?
   The Plan-Burler fasit is the decision record only, which would call a rework card unlicensed scope.
   Picked: add the highest committed round's `findings.md` to the answer key, through the fasit instructions and the rubric.
7. Does Webster-Review's range change?
   The rubric reviews the whole branch over the merge base against the plan as fasit; with only the live generation in the plan, earlier code would have no card.
   Picked: review only commits after the highest round's recorded `head_sha`.
8. Describe reads only Webster's `summary.md`; after a fresh run that is the live generation only.
   Picked: tell Describe every generation's summary.
9. Attack-surface fold-in: which stencils?
   Board names the discussion and plan rubrics; each rubric has a writer/reviewer symmetry note.
   The trigger: refusal-audit's own review wrote "goto places no restriction on targets" and nothing acted on it.
   Picked: both rubrics plus the matching writer-stencil instruction.
10. Burler focus/prior-round conflict: restate, or exempt?
    Picked: the judge writes self-contained focus entries, and the directive stencil keeps the clean-room rule unconditional.
11. Do `3613d5076` ("Still to run") and `e39909bbc` (prompt as file) stay?
    Picked: yes; both still help a crash-resumed Master within one generation.

## Rejected alternatives

- **Delete the retired cards and rely on git history alone.**
  The rework session runs in the task worktree, where the plan is not readable at HEAD; it would lose its view of what was built.
- **Archive inside `_lyx/plan/` (an `archive-gen-*` subdirectory).**
  Master runs `ls` on the plan directory and Webster-Review uses it as fasit, so the dead cards would still be in view; the plan directory should equal the plan.
- **Archive Webster's state under `WebsterDir` with a timestamp, as `--fresh` does.**
  Splits one generation's record across three places; the round directory already names the round that retired it.
- **Restart card numbers at 1 per generation.**
  No spec change, but `N:` commit subjects, coverage maps and finding citations become ambiguous across rounds.
- **Keep the append-only rule and only hide done cards from Webster.**
  Treats the prompt, not the plan; Webster's state, the batcher and the review segments would still carry every card.
- **An agent-declared review flag on each rework card.**
  The agent that writes a risky card is the least reliable judge of whether it needs review.
- **Route every rework generation through Plan-Review with no exemption.**
  Simplest, but the board explicitly keeps the exemption for text-only findings.
- **Use the overview's `approved:` flag as the skip signal.**
  A writer session could set it itself and bypass review; the skip signal must be something only Go writes.
- **A new shed outcome or conditional edge.**
  Changes a closed vocabulary in `internal/shedengine` for one row's routing; the skip seam solves it locally.
- **Exempt focus-named files from the clean-room rule.**
  Lets every focus directive reopen prior opinions before the reviewer forms its own; the clean-room rule is the method's core property.
- **Keep Webster-Review over the whole branch diff.**
  Earlier generations' code has no card in the live plan, so findings against it cannot be resolved against the answer key, and it already passed review.

## Review rounds

### Hub review (parent, 2026-10-02)

Reviewed by the hub orchestrator against the board goal before Discussion-Review, edited in place.
The record meets the goal and covers every folded-in source; no decision changed.
One addition: Open risks names the `loom-recipe.yaml` overlap with `gate-list`, which runs in parallel and reshapes the `gate` keys.

## Question ledger

- **Q1 (resolved, auto-pick)** — archive location: `round-<N>/prior-generation/`. Rationale in Interview 1.
- **Q2 (resolved, auto-pick)** — generation scope: plan, Webster run record, Plan-Review and Webster-Review run dirs. Interview 2.
- **Q3 (resolved, auto-pick)** — numbering continues via `first_card`. Interview 3.
- **Q4 (resolved, auto-pick)** — routing via `PR-Rework.on_done: Plan-Bouncer` plus a skip seam. Interview 4.
- **Q5 (resolved, auto-pick)** — exemption = every card all-`Prosa`, decided by Go. Interview 5.
- **Q6 (resolved, auto-pick)** — Plan-Review answer key adds the highest round's `findings.md`. Interview 6.
- **Q7 (resolved, auto-pick)** — Webster-Review range starts at the highest round's `head_sha`. Interview 7.
- **Q8 (resolved, auto-pick)** — Describe gets every generation's summary. Interview 8.
- **Q9 (resolved, auto-pick)** — attack-surface item in both rubrics and both writer stencils. Interview 9.
- **Q10 (resolved, auto-pick)** — self-contained focus entries. Interview 10.
- **Q11 (resolved, auto-pick)** — keep `3613d5076` and `e39909bbc`. Interview 11.
- **Q12 (resolved, auto-pick)** — in-flight runs parked at PR-Rework are restarted, not migrated, matching the recipe header's existing policy for row changes.
- **Q13 (open, for the plan)** — exact git form of the Webster-Review range that excludes parent-merge changes.
- **Q14 (open, for the plan)** — confirm the Plan-Review episode bounce budget resets on each re-entry.
