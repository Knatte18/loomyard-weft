# Support log — card-batching

## Interview

Autonomous mode: no operator answered; every point below is a self-pick recorded in the question ledger.

1. Read the board entry: the operator has already settled the cost model, hard limits, contiguous DP, profiles and calibration-first.
   Dependency `bugfix-webster` has landed (HEAD `d536a1a4e`), so Plan-Gate refuses backward dependencies and contiguous segments of the card order keep every dependency.
2. Explored `internal/batcher`: an identity-only registry, `active:` key, `LoadOrTemplate`; `Batch(cards)` takes no disk input.
3. Explored webster: every bracket verb recomputes `Batcher.Batch(plan.Cards)`.
   With identity that is stable; a batchifier that reads file sizes would regroup cards mid-run as batches change files.
   Settled: record the partition in `state.json` once per run (Decision 10), with rebaseline re-batching only the unbegun tail (Decision 11).
4. Found the plan spec's `Batchable?` column, which says Create and Edit cards never batch; no code enforces it.
   Settled: retire it, since the operator's design groups exactly those cards (Decision 12).
5. Checked the implementer stencil: it already handles several cards per fork, committing per card; no stencil change needed.
6. Explored `tools/tokencount` and the weft history: plan artifacts commits exist per slug, several per run.
   Settled: tie forks to cards by card pointers in the fork prompt; pick the last plan commit before the first fork; read git through go-git local reads.
7. Scope check against `plan-glyph-precision`, `webster-parallel-execution` and `webster-delete-ordering`: member-level grouping and parallelism stay out.
8. `ly:orch` (parent input): the batcher must take its order from the plan and not rely on `SequenceBatches`, which can still reorder; settle how the two fit.
   Settled: the batchifier's output order is authoritative and `SequenceBatches` reduces to an assertion that refuses a backward edge or cycle (Decision 12), taking over that item from `webster-delete-ordering`; the plan spec and template part stays with that note.

## Rejected alternatives

- **Recompute batches per verb from a frozen size snapshot in state.**
  Deterministic, but a rebaseline that changes one card's size can move a DP cut into a begun batch, and the existing guard would then force `reset --to start`.
- **Read sizes from a fixed git commit in every verb.**
  Same rebaseline problem, and puts git reads into the batchifier's live path.
- **Keep `SequenceBatches` reordering and rely on it being the identity on valid plans.**
  It still condenses cycles and reorders silently on a plan that bypassed Plan-Gate, splitting the cost model's segments from execution order.
- **Greedy merging of neighbouring cards.**
  Order-sensitive and not provably minimal; the operator chose the DP.
- **Tie forks to cards by `begin-batch`/`record-batch` commit times.**
  Approximate where the fork prompt names the card files exactly; kept only as nothing, not a fallback.
- **Make a cost profile the default.**
  The board keeps identity as the default; the operator switches after calibration.
- **Fit the weights automatically by regression inside the batcher.**
  YAGNI; the calibration report gives the fit, and weights are set in the profile.
- **Keep the `Batchable?` column and only batch Delete/Rename/Move cards.**
  Contradicts the operator's design, which groups small Edit and Create cards sharing files.
- **Hub-wide `batcher.yaml`.**
  Nothing asks for it, and per-worktree lets a run be tried on a profile alone.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Does the partition stay stable across a run's verbs? — Resolved (auto-pick): record it in `state.json` at first init; every verb reads it. Reason: size-based grouping is not stable against a changing tree.
- Q2 What happens on rebaseline? — Resolved (auto-pick): begun batches keep their card ids; the unbegun tail is re-batched with the active profile.
- Q3 States written before the partition field? — Resolved (auto-pick): recompute as today, so in-flight runs survive a deploy.
- Q4 Which profile is active by default? — Resolved (auto-pick): identity, per the board.
- Q5 How is a fork tied to its card? — Resolved (auto-pick): card file pointers in the fork's prompt; unattributed forks listed.
- Q6 Which plan and tree does calibration use? — Resolved (auto-pick): the last plan-artifacts weft commit before the first fork; the warp tree just before the first card commit; skip a run otherwise.
- Q7 What is the measured cost? — Resolved (auto-pick): tokencount's existing weight figure.
- Q8 What happens to the plan spec's `Batchable?` column? — Resolved (auto-pick): removed, pointing at `internal/batcher`'s doc.
- Q9 Do the implementer or Master stencils need changes for multi-card batches? — Resolved: no, they already handle several cards per fork.
- Q10 How do the batchifier and `SequenceBatches` fit? — Resolved (parent input from `ly:orch`): the batchifier's output order is authoritative; `SequenceBatches` becomes an assertion refusing a backward edge or cycle, no reordering.
- Q11 Does the profile record its estimates per batch? — Resolved (auto-pick): yes, in `state.json`, so real-run tokens can be compared with estimates.
