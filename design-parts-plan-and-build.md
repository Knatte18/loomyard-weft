# Each part agent plans and builds its own part, instead of joining the parts into one plan

`parts-plan-and-build` · Note · enhancement, loom

## Seen (plan-split benches, 2026-11-01 and 2026-11-02)

The plan-split writer (`Parts-Write` cuts the scope into parts with contracts, part writers plan each part, a chair joins them into one plan) was benched and replayed on two inputs (`docs/research/plan-split-tune-2026-11-02.md`).

- Both inputs split into 6 parts; the joined plans had 36 and 37 cards against 20 in the solo plans that landed, for the same code locations.
  About 85% of the extra cards were changes the solo plan had bundled into too-large cards; the rest were per-part overhead (PATTERN and doc cards per part, a card for one registry line), and about 9 cards were too small to be worth a card.
- Planning cost and wall time were close to the solo planner's after the tuning ($18.69 against $14.93, 24 against 25 minutes; $29.36 against $28.03, 29 against 32 minutes), from one run each.
- How well a task splits decides any gain: `warp-visibility` had a critical path of 8 of 36 cards and 15 cards on its widest level, while `run-verbs-shared` had a path of 20 of 37 cards and 119 dependencies, so its parts form one chain.
- A joined plan of about 1.75 times the cards means about as many more implementer turns, each with its fixed cost of context, tests and commit.

## Idea (operator, 2026-11-02)

Whatever is built here sits beside today's tapestry, as a new recipe or a new plan format a task chooses, and leaves today's Plan-Write and Webster unchanged: what runs today works well.
It is benched against today's course on the same inputs before it is used for real work.

- The same question can be asked of today's course: how coarse a plan Sonnet can still build from.
  A coarser plan format, with larger cards that each state the boundary they keep, moves the code reading and the choice of code locations from the Opus planner to the Sonnet implementer, which is what a part agent does.

In the long run the parts are not joined into one plan: each part agent both plans and implements its own part, which is where time and tokens may be saved.

- `Parts-Write` cuts the scope into parts with contracts, as today; then each part agent plans its part and builds it, reading only its own part and the contracts it depends on, so its context stays small; parts with no dependency between them run at the same time.
- The aim is parts small and well bounded enough that a Sonnet `high` agent plans and writes each one; Opus stays on `Parts-Write`, whose contracts carry the design.
- Whether a part agent can run on Sonnet is open: today `Parts-Write`, the part writers and the chair all run `opus[medium]` (`tapestry.yaml`), and a part writer on Sonnet was never tried.
  A first bench: the same two inputs with only `part_writer: sonnet[medium]`, checking that the parts still join with no hand edit, and at what cost.
- The split writer can answer that a task does not split, and the solo planner takes it; a task whose parts form one chain gains nothing.
- Per-part overhead cards (PATTERN, docs) are folded into the part's code cards, and cards too small to be worth a turn are merged with a neighbour.
- Open: where parts are built (one task worktree in turns, or a worktree per part), how a broken contract is caught before the parts meet, and one review of the whole diff against a review per part.
