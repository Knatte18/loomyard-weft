# Round 1 fixer report

All three findings are fixed.
Only plan files changed (cards 2 to 5); `00-overview.md` and its `approved:` key are untouched, and no git command was run.

## Fixed

- **F1 (BLOCKING)** — `02-gitrepo-has-unpulled.md`.
  Added `cmd/lyx/gitrepoboundary_test.go` to the `**Edit:**` list.
  The Intent now says `HasUnpulled` joins `gitrepoPinnedRunBoundMethods`, and that it is CLI-bound for the same reason as `HasUnpushed`.
  The ImpactSummary names the extra pinned-set entry.
- **F2 (MEDIUM)** — `03-fabric-pair-anchored-push.md`.
  The table-driven lock test now covers both `PushPairAnchored` and `PushAnchored` for the bounded-wait expiry.
  The scenario test's retry step gains a `PushAnchored` row against a records remote that declines once.
  No new test function was added.
- **F3 (LOW)** — ImpactSummary of cards 3, 4 and 5 rewritten as blast-radius conclusions.
  Card 3 states the three `PushAnchored` callers updated in-card, card 4 the one production caller moved and the one test updated, card 5 the callers gained and `Fabric.PushWeft` left with a test caller only.

## Deferred

None.
