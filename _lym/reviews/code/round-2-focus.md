---
round: 2
focus:
  - "internal/gatecli/run.go (runWithSlots, report, reportCarried, retryTier), internal/gatecli/copies.go, internal/gatecli/build.go and internal/gatecli/tierrun.go: on each path that prints a tier summary line (slotted, unslotted or inherited, carried, empty scope, shared build, --copies with its lym prebuild, rerun after an environment failure), does the printed wait equal the slot wait that the verb help and internal/gatecli/doc.go describe?"
  - "internal/gatecli/report.go, callSpan and every other reader of the RunRecord time fields: do the call span and the lym gate report figures stay correct, counting no interval twice, for new records and for records that lack a field?"
  - "internal/gateslot/runlog.go (tierSummaryPattern, tierDiscoveredPattern), read by FailedTier from internal/landingshed/verifygate.go over a verify log that the lym on PATH wrote: is a summary line read correctly both with and without the wait field, and is any other reader of the line affected?"
  - "The gatecli tests and internal/gateslot/runlog_test.go: does a test drive a real tier run for the slotted, carried, shared-build and rerun cases and assert the printed wait against a value independent of the code that computes it, and does each test assertion still add coverage?"
  - "_lym/landing/summary.md: does the diff bear out its claims about what wait measures per path, which binaries write and read the summary line, its Merge danger door and its blast radius?"
---
