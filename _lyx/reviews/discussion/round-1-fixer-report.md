# Fixer report — discussion round 1

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  Decision 4's Bound now names everything read before the first edit and changed since: the listed spans and the whole files Decision 2 reads (uncovered target and `Uses:` files, located unresolved glyphs' files).
  It also says Decision 2's Edit-only rule keeps a Write from a pre-edit read from silently dropping an earlier edit.
  Decision 2 gains the sentence that the pack-on FRESH-READ fragment tells the fork to change a file it already edited in this batch only with the Edit tool, never by rewriting it whole with Write from content read before that edit, and a `Bound: Decision 4's.` line.
  A matching acceptance criterion is added after the `plan:` criterion.
  Open risks bullet 1 now counts the whole files Decision 2 reads toward the wide first phase, and bullet 3 (now 2) speaks of a span or file made stale.
- **F2 (LOW), fixed as proposed.**
  Deleted the `kickstart-pack-measure` metric bullet from Open risks; support-log Interview item 5 keeps the lens finding.
- **F3 (NIT), fixed as proposed.**
  Removed "; see the support log" from Auto-mode assumptions bullet 1.

## Disputed

None.

## Deferred

None.
