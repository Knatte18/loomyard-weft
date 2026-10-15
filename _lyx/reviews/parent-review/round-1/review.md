---
verdict: APPROVED
---

# Alignment report — run-pane-timing at parent-review

## Scope against the board entry

The decision record covers each of the board entry's five items:

- Item 1, seconds: D1 draws every closed span to the second.
  The running row keeps whole minutes, a narrowing D1 states and grounds in the pane's redraw clearing the scrollback; every row shows seconds once it closes.
- Item 2, round totals: D2.
- Item 3, gate time and rework: D5 adds the missing record in shuttle, D6 draws in-row gates, D7 draws row-gate rework.
  The board allows a partial item 3 when today's records fall short; the design closes that gap instead, which stays within the item.
- Item 4, numbers for every shed recipe: D3 (one number per perch, one lookup over loom, darn and batten) and D4 (batten's runs overview, its only pane).
- Item 5, segment-named sessions: D8 prefixes the Bouncer and Burler roles of every review segment, Merge and Darn included.
  Scope Out leaves writers, driver, arbiter, alignment and the other single-holder roles unprefixed, because each is held by one row or stop at a time and its name is already unique in the worktree.
  The board's stated problem is review-segment sessions with identical names, and this design fixes that problem; the record decides the reading (Q7, Q8).

No work belongs to another entry:

- `report-timing` keeps the timing report.
  Scope Out leaves `lyx report` and `lyx loom status` alone, and the new gate spans become a record that note can read.
- `loom-model-key-names` is untouched: D8 adds a role prefix, not a model key.
- `burler-fixer-fork` is unbuilt, and D8 binds only the sessions that exist today.
- `viewer-off-shell` and `reed-status-buttons` change reed's focus and status bar; D8 needs no reed change.
- `run-verbs-shared` is unaffected: `lyx loom checklist` keeps its name.

## Overlap with runs in flight

`lyx loom peers` lists no parent commits and one peer.

- `refusal-route-config-empty` (row 4; this run's own row is 3).
  - What overlaps:
    - Both change `contracts/recipes/loom-recipe.yaml`. The peer adds Webster's `on_reroute: Plan-Burler-Review` and new text in Plan-Burler-Review's instructions; this run adds `role_prefix` to the review-segment rows.
    - Both change the Webster shed adapter in `internal/shedadapters`. The peer adds a reroute on the stuck arm; this run copies the Merriam run's gate count and spans into the history entry.
    - The peer's reroute history entry is one of the entry kinds that D5 gives gate spans to.
  - None of these conflicts:
    - The overlaps sit at different places in the files.
    - D7 attributes only Stuck entries with `routed_to`, and a reroute entry has outcome `reroute`, so the peer's backward jump is never counted as gate rework.
    - The peer's open risk already says that whichever run lands second renders Webster's reroute into Plan-Review.
  - Landing order: the peer is further along (row 4 against 3), so it lands first and this run takes its changes at Merge-In.

No bookkeeping file overlaps in a way worth naming.
