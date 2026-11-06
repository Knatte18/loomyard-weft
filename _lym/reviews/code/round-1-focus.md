---
round: 1
focus:
  - "internal/dobbyengine/waitreminder.go: does the wait start equal the later of dobby's first read at the row with no covering pause and the latest covering resumed entry's `resumed_at`, and is it cleared when the child leaves the row, stops running or a pause covers it?"
  - "internal/dobbyengine/waitreminder.go: is the start recorded whatever the presence, while posting happens only under `slack`, and does a return to `slack` remind at once of a wait already past the interval?"
  - "internal/dobbyengine/waitreminder.go: is the reminder cadence one line per `remind_after_min` interval, with the last-reminder time in memory, and does a post creel refuses log at Warn and retry at the next check without advancing the last-reminder time?"
  - "internal/dobbyengine/waitreminder.go and loop.go: are only PR-Review and Sampler-Session reminded, with Beam-Await excluded, and does the check run once a minute in every worktree's dobby, the prime's included?"
  - "internal/dobbyengine/state.go: is the state record's new field tolerant JSON, so an old or malformed record still loads?"
  - "internal/beamcli/dobbydeps.go and internal/pauserecord/record.go: does the latest-resumed-entry lookup pick the right entry per slug, including a slug with no resumed entry or a pause still open, and are the presence reader, interval loader and SLACK mailbox post wired correctly?"
  - "internal/slackengine/config.go and template.yaml: does `remind_after_min` default to 30, and are invalid values (zero, negative) handled?"
  - "Test economy across waitreminder_test.go, dobbydeps_integration_test.go and record_test.go: is any test duplicated by another, and is each added behavior covered, including the acceptance criterion's paused-and-resumed case with and without dobby reading the pause?"
  - "Docs: do docs/overview.md and the changed doc.go files carry what the diff changes, and does the commit message or doc text claim anything the diff does not bear out?"
---

Seed from the decision record's D8 and acceptance criterion; the range is two commits over 15 files.
