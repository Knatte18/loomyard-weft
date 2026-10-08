# Log the silent overview-frame fallback in rebaseline

`overview-frame-fallback-log` · Note · enhancement, webster

## Seen

The PR-Gate review of `webster-record-batch` (PR #473) found that `recordedOverviewFrame` in `internal/websterengine` returns an empty frame without logging when the stored baseline copy has no parseable Card Index, so the reason a rebaseline refuses on an old record is invisible.

## Idea

Log the fallback at Info with the baseline path, and name it in the rebaseline refusal.
