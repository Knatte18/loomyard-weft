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
   Settled: retire it, since the operator's design groups exactly those cards (Decision 13).
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
- Q6 Which plan and tree does calibration use? — Resolved (auto-pick): the rule in Decision 3; skip a run otherwise.
- Q7 What is the measured cost? — Resolved (auto-pick): tokencount's existing weight figure.
- Q8 What happens to the plan spec's `Batchable?` column? — Resolved (auto-pick): removed, pointing at `internal/batcher`'s doc.
- Q9 Do the implementer or Master stencils need changes for multi-card batches? — Resolved: no, they already handle several cards per fork.
- Q10 How do the batchifier and `SequenceBatches` fit? — Resolved (parent input from `ly:orch`): the batchifier's output order is authoritative; `SequenceBatches` becomes an assertion refusing a backward edge or cycle, no reordering.
- Q11 Does the profile record its estimates per batch? — Resolved (auto-pick): yes, in `state.json`, so real-run tokens can be compared with estimates.
- Q12 What does the DP do for a card over `budget` on its own? — Resolved (auto-pick): a one-card batch is always feasible; `budget` and `max_cards` bind only multi-card batches (Decision 6).
- Q13 Which weights does the calibration report use, and who sets the template's? — Resolved (auto-pick): the report takes a named profile's coefficients; a late card sets the template weights by one scale factor from the report's median ratio.
- Q14 What does a mid-run `batcher.yaml` error do to a recorded partition? — Resolved (auto-pick): nothing; the verb refuses before touching state, and only a successful first init or rebaseline replaces the partition (Decision 8).
- Q15 Which batchifier does a state without a recorded partition use? — Resolved (auto-pick): identity, whatever profile is active (Decision 10).
- Q16 Where does merging save cost in the model? — Resolved (auto-pick): a per-fork message count paid once per batch (Decision 5); without it every merged card costs at least what it costs alone. The operator should confirm this term matches the accepted model.
- Q17 Which card owns the coefficients type and its parse, given the calibration report precedes the profiles? — Resolved (auto-pick): the estimator's card (Decision 1); Decision 8 keeps the registry, `active:` resolution and load errors.
- Q18 In which unit are `alone_above` and `budget`, and who sets them after calibration? — Resolved (auto-pick): the unit of their profile's coefficients, context tokens initially; the calibration card scales the three context coefficients by the median ratio and multiplies both thresholds by the same factor, which puts all three in tokencount's weight unit (Decision 5, Auto-mode assumptions).
- Q19 When is a fit poor? — Resolved (auto-pick): the 75th-over-25th percentile spread of per-card ratios above 3; the card then commits nothing.
- Q20 What does a past card with several forks or none show in the calibration table? — Resolved (auto-pick): summed weight, or skipped (Decision 3).
- Q21 What does a rebaseline do on a state without a recorded partition? — Resolved (auto-pick): keeps identity for the tail and records none (Decision 11).
- Q22 What is an invalid profile parameter? — Resolved (auto-pick): a negative coefficient, a non-positive threshold, or `max_cards` below 2 (Decision 8).
- Q23 What profile name and estimate do batches carry after a rebaseline under a different active profile? — Resolved (auto-pick): each batch records its own profile name and estimate; a begun batch keeps both and the re-batched tail gets the then-active profile's (Decisions 8, 10, 11).
- Q24 Which card places the initial coefficients, and where does the tuning card sit? — Resolved (auto-pick): the estimator's card holds the coefficients type and its parse, the profiles card ships the template's initial coefficients after the calibration report, and the tuning card follows both (Decision 1).
- Q25 What is a card's estimate in the calibration table? — Resolved (auto-pick): its one-card batch cost under Decision 5, from the one segment-cost function the live batchifier also calls.
