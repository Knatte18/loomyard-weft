```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for consistency with its surroundings (lens fix-consistency).

**Findings**

1. **LOW**: the `decisionEntries` doc comment is stale. The text is `internal/loomcli/decision.go:123`: "A --from call refuses any per-entry flag beside it and a file that is unreadable, not one JSON array of objects, or empty."
   - The fix (F1, F2) added two new refusals: an unknown key in an entry, and data after the array (a stray `]` now included).
   - The comment's list stops at unreadable, not-an-array and empty, so it no longer covers every refusal the function makes.
   - The twin messages agree. `decisionWayFromFile` (line 82) already names the allowed keys, and the help text at lines 891-896 lists the keys.
   - The help does not say that an unknown key is refused. That is not required by the finding.

2. **LOW**: `append.go` has two header comments that are no longer true.
   - Line 1: "append.go implements AppendDecision, one of the two write paths into the decision record." The file now also holds `AppendDecisions`, and `AppendDecision` is a one-line wrapper over it.
   - Lines 3-4: "AppendDecision adds one design call…". This is still true for the wrapper.
   - Lines 40-56 of the `AppendDecision` doc describe the write, check and restore in full, which is now `AppendDecisions`' behaviour.
   - The fixer touched only the `AppendDecisions` comment and `doc.go`. `doc.go` line 4 now names `AppendDecisions` correctly.
   - This is minor and was not part of F6. Its wording mismatch with the file header is the only inconsistency.

No other twin was left behind. The `AppendDecisions` doc now states the behaviour directly (F6), the add help says `--from` replaces the required flags (F5), and the new test rows match the refusal text ("unknown field", "data follows the array").

**Tool calls**
1. One Bash call: grepped `decision.go` for "unknown", "JSON array" and the `decisionWayFromFile` text, read the add help lines 884-900, and grepped for other mentions of `AppendDecision`. My context held only the fixer's edits, not the surrounding doc comments and help text.
