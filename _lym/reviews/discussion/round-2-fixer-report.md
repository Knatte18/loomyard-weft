# Fixer report — Discussion, round 2

Every finding of the review was fixed in `_lym/discussion/decision-record.md`.
The support log needed no change.

## Changes

- **F1 (BLOCKING)** — fixed as proposed.
  - D8 refuses unless the run's state is running and its current producer is `Parts-Plan-Write`, read from the run status and the recipe's rows.
  - D8 refuses unless the caller is the chair's strand. The caller is resolved by the creel daemon, as it resolves a sender, and compared against the row's seat table.
  - D8 lists every refused caller with its way forward.
  - The part-fix refusal now says the chair has the writer fix the part, or fixes it itself (D9).
  - The Bound sentence is rewritten.
  - D11 forbids part writers to run `join-parts` and `insert-card`.
  - The D8 criterion gains refusals for a blocked run and for a caller that is not the chair.
  - Self-check changes: the halted-run way forward is now `lym loom resume` alone, and states the live-chair case. The caller-identity limit off Linux is stated.
- **F2 (MEDIUM)** — fixed as proposed.
  - D6 maps a writer that timed out to Stuck naming the seat, its part and the timeout.
  - D6 maps a writer that failed to start to an error wrapping `ErrNotStarted` with its `StartError`, and a writer that died to an error wrapping `ErrAgentDied` naming the writer seat.
  - `MultiLLMProducer` reads the ended seat's `SeatResult` before the chair's result.
  - The D6 criterion is extended to cover these cases.
- **F3 (MEDIUM)** — fixed as proposed.
  - D5's resume line now tells the chair that the writer was stopped and that any fix asked of it is lost. Per the self-check, this replaces the earlier "part stands" line rather than adding to it.
  - D9 keeps a part whose fix request was outstanding at a resume open: the chair rechecks it and fixes it itself before the join.
  - D12 lets the chair read the code of that part's area when it fixes the part itself.
  - The D5 criterion is updated.
- **F4 (MEDIUM)** — fixed differently.
  - The D2 key list gains `parts_chair_timeout_min`. When unset it means twice the run's effective `plan_timeout_min`, and a set value not above `plan_timeout_min` is refused when config loads.
  - The proposed literal default, twice `plan_timeout_min`'s default, was not used. A profile that raises `plan_timeout_min` would leave the chair shorter than its writers again (fix-proposal lens, `internal/loomengine/tapestry.go:33`).
  - D6 states the chair-timeout precedence. The proposed sentence "`EndedStep` wins" does not hold at HEAD, because `join` lets a chair that has also ended decide (`internal/seatengine/engine.go` `join`). D6 now says that for a writer table `join` marks the ending writer `EndedStep` even when the chair ended first or together with it.
  - The D2 and D6 criteria are updated.
- **F5 (MEDIUM)** — fixed as proposed.
  - D3 adds the read-only `lym loom validate-parts`, which calls the `parts` gate's own check function and refuses by the recipe's rows.
  - D10 runs it and fixes every finding.
  - The verb is added to the PATTERN-loom-contract and PATTERN-refusal-way-forward constraint lines and to the D3 criterion.
  - Self-check change: the refusal's way forward names `lym loom validate-plan`.
- **F6 (LOW)** — fixed as proposed. D4 commits on any non-empty pointer, and the D4 criterion covers an exhausted parts gate committed with a clean tree.
- **F7 (LOW)** — fixed as proposed. The first Open-risks bullet is deleted.
- **F8 (LOW)** — fixed differently.
  - D9's bound is "never again once it has written anything into the plan directory after the join", and the seam sentence matches it.
  - The proposed "once its first shared-spot card is in" was not used. A task with no shared spots gets no such card, so a rejoin would then wipe the chair's framing and `## PATTERN changes` (fix-proposal lens; D3 allows zero shared spots).
- **F9 (LOW)** — fixed as proposed. D10 gains the `parts_max` and split-note bullets, and the D10–D12 criterion names `parts_max`.
- **F10 (NIT)** — fixed as proposed. The Open-risks line names the signature-lookup allowance.

## Self-check

- fix-claims, MEDIUM, "`EndedStep` wins" is false when the chair has also ended — fixed with own change: D6 changes `join` for writer tables.
- fix-claims, LOW, caller identity resolves by pane only on Linux, and no path to it is named — fixed: D8 names a new creel daemon request and states the off-Linux limit.
- fix-consistency, MEDIUM, D5 gives two lines for one case and the criterion repeats both — fixed. This is merged with fix-proposal 2.
- fix-consistency, LOW, the D2 key list omits the new key — fixed: the key is added to the list.
- fix-consistency, LOW, D6 "a failed writer … leaves the run Stuck" is too broad — fixed: the sentence names the Stuck cases and the error cases.
- fix-way-forward, LOW, the `validate-parts` refusal names no action — fixed: it names `lym loom validate-plan`.
- fix-way-forward, LOW, goto before resume is redundant for a run already at the row — fixed: the way forward is `lym loom resume`. This is merged with fix-proposal 4.
- fix-proposal, MEDIUM, the F8 bound fails with zero shared spots — fixed with own change: the bound is "anything written after the join".
- fix-proposal, MEDIUM, D5 contradiction — fixed. This is merged with fix-consistency MEDIUM.
- fix-proposal, MEDIUM, the literal chair-timeout default brings F4 back when `plan_timeout_min` is raised — fixed with own change: the default is derived and a value not above `plan_timeout_min` is refused.
- fix-proposal, LOW, the way forward claims a fresh spawn for a paused run whose chair is live — fixed: the way forward states the live-chair case.

## Disputed

None.

## Deferred

None.
