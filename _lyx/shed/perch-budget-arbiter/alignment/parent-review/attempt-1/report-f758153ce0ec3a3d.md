---
verdict: APPROVED
---

## Scope check against board entry `perch-budget-arbiter`

Darn review budget:

- Budget keys become ordinary keys, minimum 1, settable through `run_settings`: D2.
- A separate guard key switches Darn-Review off: D3 (`skip_review`), with D4 migrating an old `review_max_bounces: 0`.
- "Check whether darn and loom can share one config shape": D1 shares it in `internal/reviewbudget`, below both engines.

Arbiter spawns only when needed:

- Case 1, hard cap or circling only: D5 narrows awaiting eligibility to `hard-cap` and `circling` and drops the arbiter stencil's `budget` rule, as the entry's "Changes this implies" asks.
- Case 2, a `blocked` stop naming a way forward is refused: already landed in `30385e1ae` (present on this branch); D6 keeps it.
- Case 3, and the entry's open point on it: D6 keeps the spawn behind the fork or tackler and gives its reason.
- The entry's first open point, the Merge-In verify goto: Scope Out answers it as a driver-run goto, with the verb refusing that stop.
- Twice-failed transient to the fork first: D7.
- The soft-cap and no-BLOCKING table rows: already settled by `072e2ed86` (judge-cap-call), which the support log checked.
- The awaiting `parent_notice` row: D8 keeps the stencil calling the verb and lets the verb refuse, instead of narrowing stencil line 109.
  The entry's ask, no arbiter spawn and the stop going straight to the orch, still holds, since a refused call spawns nothing.
  The entry presents its body as a proposal to design against, D8 records the departure and its reason, so this is a design choice for Discussion-Review, not a scope deviation.
- Double notice: D9 holds batten's notice until the arbiter has handed on, and keeps the orch's check-before-acting guidance the entry names.

Out of scope as the entry frames them: "Later: batten stops" and the general helper are marked as the operator's open thinking; counting arbiter spawns belongs to `report-timing`, and the design keeps the records it reads.
No goal or Done-when point is left out, and no work belongs to another entry.

## Board entries touching this task

- `report-timing`: counts arbiter spawns from run records; this run changes no record format. No duplication.
- `orch-split`: a note about splitting the orch and driver; no decision here contradicts it.
- `overview-seats`: covered below as a peer.
- `model-downgrade`, `codex-engine`, `discussion-multi-llm-bench`: mention darn or the arbiter in passing only.

## Overlap with peers and parent commits

The parent has no landed commits listed.

- `overview-seats` (row 5, Discussion-Burler-Review; own row 3).
  Shared files: `internal/loomcli/doc.go`, `contracts/specs/refusal-spec.md`, `internal/configreg`, the `loomengine` config `Categories` map, and darn's routing area, where it moves Darn-Write onto a seat table.
  Its config changes touch other keys (`discussion_producer`, `plan_producer`, `discussion_advisors`, the lens keys) and loom's `MigrateConfig`; this run touches the review budget keys and darn's own migrate hook.
  Its checklist reads arbiter attempt records, which this run leaves unchanged.
  The two runs make no conflicting decisions and change no PATTERN entry differently: a text overlap only, which this run's Open risks already name.
  Landing order: `overview-seats` is further along (row 5 against row 3), so it lands first and this run takes its changes at Merge-In.
- `hibernate-trigger` (row 7): a new hibernate module; no overlap.
- `twb-f2-ia`, `twb-f2-if`, `twb-f2-pa`, `twb-f2-pf` (row 8) and `twb-f2-wa`, `twb-f2-wf` (row 3): bench pairs replaying `intake-bugs-1012`, a plan-gaps task and `run-window-bugs`; they cite the arbiter's unchanged `status or envelope unreadable` refusal only. No overlap.

Bookkeeping: the new `pattern/PATTERN-leaf-packages.md` row for `internal/reviewbudget` is a shared index line and does not escalate.
