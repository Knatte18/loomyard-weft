---
round: 2
focus:
  - "internal/battencli/dobbyreads.go, the landed-commit read closure inside DobbyRuns: settle whether any error from shedrun.ReadLandedCommit other than a missing file (a directory at the path, a permission or I/O error) still lets the run list as landed with an unknown commit and the other runs be listed and driven, and whether a test covers that row."
  - "internal/landingshed Finalize.Call, both recordLanding call sites (after the first parent-side merge and after the merge-in retry): settle whether the recorded landing commit and its order relative to the mark and the push are asserted by a test on each site, so that deleting either call fails a test."
  - "internal/dobbyengine landNotices and its test: settle whether the Requested value refreshed while the entry is on the board, and the kept reading after a failed board read, are each pinned by a test row."
  - "internal/boardengine notify_on_land: settle whether a test pins that setting the field leaves the revised date unchanged and that board replay carries the field."
  - "internal/loomcli landingdeps.go RecordLanding and the beamcli ClearNotifyOnLand closure: settle whether a test ties the write path of the landed-commit file to the path battencli.DobbyRuns reads, and whether the clear wiring is exercised."
---

The entries name sites to re-check in the committed diff and the question to settle at each.
