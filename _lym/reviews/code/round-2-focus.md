---
round: 2
focus:
  - "internal/dobbyengine/landnotice.go, `flagMembers` and `landNotices`: settle whether a member whose run is listed past Land, or whose slug has landing-notice memory, can have its notify-on-land flag set again after the landed line cleared it, and whether any path then posts a second landed line for one landing."
  - "internal/dobbyengine/landnotice.go, `landNotices` and `rememberRuns`: settle whether a member that lands, with its board entry reading done, in the tick the beam's pull request is first recorded still gets exactly one landed line carrying the follow-up, and whether the flag clear stays owed only while the entry carries the field."
  - "internal/beamcli/dobbydeps.go, the `PullRequestRecorded` seam closure in `hubDobbyDeps`: settle whether tests in internal/beamcli/dobbydeps_integration_test.go cover a hub beam with no landing record, a record with no pull request, and a record with a pull request, each with a nil error."
  - "internal/dobbyengine/landnotice_test.go, `TestLandNotices`: settle whether each added behavior has a row that fails if it breaks, namely the member filter (other project, done status, kind), the failed-write retry of `SetNotifyOnLand`, the follow-up on the commit-unknown line and on a done entry's line, and `land_notice: off` with a recorded pull request."
---

Focus targets the code paths changed since the first review of the land-notice diff; the base review still applies to the rest of the range.
