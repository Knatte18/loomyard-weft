# Fixer report — discussion round 1

## Changes

- **F1 (fixed as proposed).**
  Decision 1 now ends a turn at a user line whose text starts with `[Request interrupted by user`, as well as at an assistant line with a non-`tool_use` `stop_reason`.
  The Auto-mode assumption and the Open risk are narrowed to a crash that leaves no marker, and an acceptance criterion covers the marker.
  Premise checked: 78 task-run transcripts outside the prime's project directory carry the marker.
- **F2 (fixed as proposed).**
  Decision 4 states that the union covers the run's worktree transcripts and their sub-agents only, and that the orch session and every orch sub-agent stay out, including one charged wholly to the run, so parent review is idle.
  Decision 7 states that "All runs" sums per-run times, idle and spans, with no union across runs.
  An acceptance criterion covers orch exclusion.
- **F3 (fixed with the proposal's second option, framed differently).**
  Decision 8 now names what the widened default lets through (a slug read against the current hub's weft when `-hub` names another hub) and its bound: a `-hub` other than the current worktree's parent with `-weft` empty is refused, naming `-weft`.
  The resolvable-worktree failure moved from Open risks into the Decision.
  The proposal's first option, applying the default only when `-hub` is defaulted, was not taken: a named-slug run with `-hub` set would then silently get no status, which is the silent failure the finding names.
  Both options are recorded in the support log's Rejected alternatives and Question ledger.
- **F4 (fixed with own change).**
  Decision 3's span start, from `started_at` or the earliest transcript line, is now bounded: never earlier than the commit date of the slug's next-older archive tag, never later than the first history entry's end.
  The proposal's example bound, "the earliest line of a session still active at the first history entry's end", was not taken: the Discussion writer's session ends seconds before the first entry's `at`, so it would fall outside "still active" and the run's own opening work would leave the span.
  The older-tag bound uses data the tool already lists in `finishedRuns`.
  The span end is now the latest end among entries, and Decision 5 gives each entry the interval from the latest earlier end, empty when its own end is not later, so elapsed sums to the span when entries are out of order or `last_at` precedes `at`.
  Open risks bullet 3 now rests on the bounded start, and a new bullet names the in-flight reused-slug case the newest-tag rule cannot bound.
  Acceptance criteria are updated to match.
- **F5 (fixed as proposed).**
  Decision 9 names one shape: a run-id-relative path pair for `seed.json` and `status.json` that validates with `ValidateRunID`, applies no alias or legacy `self` fallback and joins the same `shed` segment declaration, plus a strict decoder over seed bytes.
  An invalid run id, or a seed or status that fails to decode, aborts the report naming slug, tag and file; an acceptance criterion covers it.
- **F6 (fixed as proposed).**
  Auto-mode bullets restating Decisions 2, 4, Scope and 10 are cut.
  Decision 6's rationale is reduced to the recorded finish time per row; the counterfactual stays in the support log's Rejected alternatives.

The support log gains a round 1 entry under Review rounds and Question ledger rows for the new auto-picks.

## Disputed

None.

## Deferred

None.
