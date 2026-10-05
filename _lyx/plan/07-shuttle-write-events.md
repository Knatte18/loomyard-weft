# Card 7 — shuttle-write-events

**Edit:**
- `internal/shuttleengine#ForkAudit`
- `internal/shuttleengine#ForkReport`
- `internal/shuttleengine/claudeengine#transcriptLine`
- `internal/shuttleengine/claudeengine#transcriptBlock`
- `internal/shuttleengine/claudeengine#auditParentTranscript`
- `internal/shuttleengine/claudeengine#auditForkTranscript`
- `internal/shuttleengine/claudeengine#Claude.AuditForksIncremental`
- `internal/shuttleengine/claudeengine/audit_test.go`

**Create:**
- `plan:internal/shuttleengine#WriteEvent` -> `type WriteEvent struct`
- `internal/shuttleengine/claudeengine/testdata/parent-writes-results.jsonl`

**Intent:**
D5 and D6 need, per Write, Edit or NotebookEdit call, when it happened and whether its tool result succeeded; the audit facts today carry only the path.

In `internal/shuttleengine/forkaudit.go`, add `WriteEvent` with `Path string`, `At time.Time` and `Succeeded bool`.
`ForkAudit` gains `ParentWriteEvents []WriteEvent` and `ForkReport` gains `WriteEvents []WriteEvent`, both in transcript order, beside the existing path lists, which keep their meaning for the audit that reads them.

In `claudeengine/audit.go`:
`transcriptLine` decodes the line's top-level `timestamp`, and `transcriptBlock` decodes a `tool_result` block's `tool_use_id` and `is_error`.
`auditParentTranscript` and `auditForkTranscript` remember each write `tool_use` by its id with its path, and resolve it on the `tool_result` that names that id: `At` is that result line's timestamp, `Succeeded` is true only when the result exists and `is_error` is not true.
A write with no result in the transcript is an event with `Succeeded` false and `At` its `tool_use` line's timestamp.
The fork's spawn-boundary exclusion applies to events as to counts.
`AuditForksIncremental` threads the parent events into `ForkAudit`.
The transcript shapes stay in `claudeengine` (PATTERN-shuttle-provider-seam).

Tests on a new `testdata` transcript holding a succeeded Write, an Edit whose result is an error, and a Write with no result:
the parent audit yields three events in order with the right timestamps and `Succeeded` flags;
a fork transcript yields the same through `ForkReport.WriteEvents`;
the existing path lists are unchanged.

**ImpactSummary:** two audit value types gain an event list each, filled by the existing transcript walk; every current reader keeps its fields.
