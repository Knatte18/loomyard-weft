---
round: 1
focus:
  - "internal/logger/retention.go sweep(): check the grouping, live-group exemption, age cutoff and count ranking against card 1's rules — including whether a group mixing a live-pid file with dead-pid files is kept whole, and whether a stat failure falls back to the filename timestamp without aborting the sweep."
  - "internal/loggerconfig/config.go Load(): settle whether time.Duration(days) * 24 * time.Hour can overflow for a large positive trace_retention_days, and if it can, what cutoff sweep() then computes and which non-live groups it deletes."
  - "cmd/lyx/exitsweep.go notifyExitAndSweep and exitSweepBounds: confirm the D3 order holds on every path (NotifyExit, arm check, no config load before the arm check, the redirected branch skips Load), and whether the Warn emitted after arming reaches both the trace file and the command's output the way card 5's integration test expects."
  - "internal/logger/sink.go sinkDir, sinkAnchorPath and CurrentSinkArmState: check that every write and read happens under sinkMu, that resetDurableSinkLocked clears both, and that Armed = sinkArmed && sinkOK matches card 4's four test cases, including a redirect after an arm."
  - "Tests in retention_test.go, sink_test.go, exitsweep_test.go, config_test.go, configsync_test.go and main_integration_test.go: check that each case card 1–5 lists exists and actually exercises its claim (mtimes set by os.Chtimes, invalid fixtures change only the key under test, the --help runs cover both the absent and the invalid logger.yaml), rather than passing vacuously."
  - "Comment convention per CONSTRAINTS.md's Comment Line-Break Convention: check new and changed Go comments (retention.go header and Sweep doc, sink.go resolveSinkDirLocked, isLyxWorktree and SinkArmState docs, exitsweep.go header, loggerconfig package and function docs) for semantic line breaks, one sentence per line with breaks only at clause boundaries; a line-width finding is never legitimate."
  - "Per-card mechanical checks: for each card's Edit group, confirm the ImpactSummary's blast-radius claim holds in the tree (Sweep's and resolveSinkDirLocked's callers, configreg.Modules' reach into fabric clone and hubforge tests, configStrictnessDegradingSet and CONSTRAINTS.md's degrading set); for each Create group, confirm nothing equivalent already existed that the new symbol duplicates."
  - "Docs that describe the sweep (internal/logger/logger.go's durable-sink section, docs/shared-libs/README.md, and any manifest/designs or docs page): check that none still states the retired rules — sweep at sink open, filename-timestamp ranking, or a 50-file count."
---

The diff spans 20 files across five cards, but the risk sits in three places:
the rewritten sweep, which deletes files;
the exit hook, whose ordering keeps zero-exit runs from arming the sink;
and the config loader, whose validated integers become a deletion cutoff.
The Edit/Create cards carry blast-radius and nothing-equivalent checks rather than a Rename or Delete mechanic, so the mechanical-check entry names those obligations.
CONSTRAINTS.md sets a semantic-line-break rule for Go comments and forbids width findings, so the comment entry asks only about break placement.
