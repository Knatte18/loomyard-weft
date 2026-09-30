# Round 1 fixer report — discussion

All seven findings fixed; `lyx loom validate-discussion` exits 0 after the edits.

## Changes

- **F1 (BLOCKING)** — D3 now fixes the exit order: `NotifyExit`, then the arm/redirect query, then no load and no sweep for an unarmed sink, template defaults for a redirected one, and a `logger.yaml` load followed by the sweep otherwise.
  It states why: `LoadOrTemplate`'s degrade record is Info and would arm the sink.
  D3's rationale no longer implies an earlier load.
  D4 adds the logger's arm/redirect query and points at D3's order.
  D5's rationale rests on that gate.
  Constraints gain "the retention config load adds no arm trigger".
  Acceptance criteria gain the case of a never-armed process with an absent or invalid `logger.yaml`.
  support-log.md's Question ledger gains Q9 recording the resolution.
- **F2 (LOW)** — The Goal's second sentence now states the bound the decisions give (outranks older-active traces, survives until N newer-active traces exist or the age bound passes) instead of an absolute guarantee.
- **F3 (LOW)** — D6 deleted. Its constraint remains in Scope, and its deliberation remains in support-log.md's Interview item 5 and first Rejected alternative.
- **F4 (NIT)** — D1's rationale names the children grouping serves (direct-exec children inheriting the trace id) and points tmux-spawned sessions to D2.
- **F5 (NIT)** — Windows `proc.IsAlive` bullet removed from Open risks.
- **F6 (NIT)** — The operator-surface Auto-mode assumption removed; the Open risk carries it.
- **F7 (NIT)** — The reconcile bullet removed from Open risks; Scope and an acceptance criterion carry the requirement.

## Deferred

None.
