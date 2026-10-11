---
round: 1
focus:
  - "Land producer and row (cards 8, 9): does every failed Land step, a seam read error included, end as a Stuck naming the step, per the land-steps-stuck decision, and does the pause path match the land-row-pause decision?"
  - "Run markers (cards 7, 8, 10): is every one-run marker named in battenshed's `runMarkerSuffixes` and removed by `ClearRunMarkers` where a fresh status is seeded?"
  - "Publish retry (cards 3, 10): does Run-Shed retry only the first Publish verify failure of a run, send it to the record's target producer, and stop goto and resume retries at the 30-minute bound?"
  - "Push retry (cards 1, 2): do the plain push primitives retry both a remote `(failed)` and `cannot lock ref`, wrap an exhausted retry in its own sentinel, and do all of Add's branch pushes and the archive tag push use it?"
  - "Notices (cards 6, 7): is the `done` notice gone from Run-Shed, and does the row notifier notify once per failure episode with at most `noticeMaxAttempts` tries 30 seconds apart?"
  - "Test economy: do the new tests duplicate coverage an existing test already gives, and is any behavior the diff adds left uncovered?"
---

Seed focus drawn from the plan's cards and shared decisions; the reviewer still judges the whole committed diff.
