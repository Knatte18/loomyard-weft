# Card 20 — status-spec-docs

**Prosa:**
- `contracts/specs/loom-status-spec.md`
- `docs/overview.md`

**Intent:** The status contract and the overview follow D1–D4 and D8.

`loom-status-spec.md`: the `error` key's blocked-halt description says an exhausted budget's reason carries the told resume clause, not a goto; the history paragraph says a `goto` entry no longer ends an episode, a `budget_capped` Stuck ends it unless a goto entry directly follows, and a `generation_start` Stuck is counted and ends it; the schema gains `pause_reason`, `pause_requested_by`, `pause_before_by` and `pause_after_by`, with their writers and when each is cleared.
`docs/overview.md`'s darn paragraph keeps `lyx loom goto --to Darn-Write` as a re-entry, and any sentence that pairs a goto with a fresh budget or a halt with a reflection follows the new rules.

**Commit:** 20: docs: the status contract and overview follow the re-entry rules
