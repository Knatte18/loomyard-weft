```yaml
lens: fix-consistency
outcome: reported
```

**Findings**

1. **LOW**: two comments still describe the condition-stop reason as "the stop report alone". F3 changed it.
   - Changed text: the `conditionStopped` branch in `internal/loomshed/darnwrite.go`.
   - Stale text 1: `internal/loomshed/darnwrite.go:110`, in the `Call` doc.
   - Stale text 2: `internal/loomshed/doc.go:91`.
   - Both are false for an empty report.

No other mismatches: the only non-test caller of `fronttask.File` is `internal/adoptcli/fixloop.go:258`; the overview paragraph agrees with `internal/dobbyengine/doc.go:100-106`; the `File` doc and package doc agree.

Tool calls: grep for the stale phrases and `startAction`; grep for `fronttask.File(` callers and `on_land` in the overview; `sed -n 100,112p internal/loomshed/darnwrite.go`.
