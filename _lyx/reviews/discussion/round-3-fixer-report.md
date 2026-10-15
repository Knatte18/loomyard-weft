# Fixer report — discussion round 3

## Changes

- F1 (fixed as proposed): the Scope Out bullet now gives the follow-up's capability rather than `tools/forkaudit` names.
  The audit tool of `glyph-audit-tool` learns the per-card layout, tells the layouts apart by prompt and by `pack_layout=card`, and reads spans from the `card-pack` tool results with the card pack trace lines as the cross-check.
  The real constraint stays as capability wording: the per-card prompt check runs before the file-layout check.
  The `depends_on` sentence is kept.
  The Open risks bullet now reads "Until the audit tool learns the per-card layout, every per-card fork classifies as a file-layout fork, with a trace disagreement."
  Premise checked: `tools/` holds no `forkaudit`.
- F2 (fixed as proposed): D3's Bound no longer claims the durable sink is the only write path.
  It says the Info line goes only to the trace file, and the Warn also prints to stderr because the logger's stderr half admits Warn at its default threshold, so the fork's tool result carries it beside the stdout markdown.
  Premise checked: `internal/logger/logger.go` builds the stderr handler on `levelVar`, which has a default Warn threshold, and the dual handler fans every record out to both halves.
- F3 (fixed as proposed): D6 now reads "each `card-pack` call that renders a pack".
  The acceptance criterion says a refusal logs no card pack Info line.
  D3's Bound adds that a refusal exits non-zero, which opens the trace file with its header and leaves no Info line in it.

## Disputed

None.

## Deferred

None.
