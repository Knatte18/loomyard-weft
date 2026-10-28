```yaml
lens: fix-consistency
outcome: reported
```

1. LOW: internal/shedverbs/loop.go:57, the LoopSpec.LoopTraceFile field comment still says the file is the trace file of a stop whose child never started, while the fallback at loop.go:309 now applies to any interrupted stop with an empty trace file.
2. LOW: internal/shedverbs/doc.go:38-39 states the interrupted stop's trace file is the child's first trace file or its captured stderr, without the loop's own trace file fallback that line 42 adds.

Checked, consistent: the start-failed after <n> tries form across loop.go, doc.go and the test; no remaining startFailed reader; one startChild caller; LoopTraceFile set in looparm.go and arm.go.

Tool calls: 2 greps over the repo for the start-failed, LoopTraceFile and startFailed mentions.
