```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| D1 stamp command on `PostToolUse`, `PreCompact` and `PostCompact` | `TestBuildSettings_RecordingHooks` (`settings_test.go`, incl. apostrophe escaping), plus the `settings-golden.json` rewrite |
| D1 creel mail hook stays on `PostToolUse` after the stamp, in a hub and outside one | `TestBuildSettings_CreelMailHook` and `TestPrepare_CreelMailHookFollowsTheBoardDirectory` |
| D2 newer stamp wins as `LastActivity`; an older stamp leaves the transcript time | The two new `TestReadAgentActivity` rows. The missing-stamp case rides every existing row. |
| D3 a waiterless running run is read with API-error and usage-limit parts zero | The `a running run whose waiter is gone…` row (`activity_test.go`) |
| D3 waiterless episode logged once | The episode test in `TestReadAgentActivity`, with its message and counts updated |
| D3 a finished or stopped run is skipped; dead-pid and pid-less running runs are read | The `a finished run is skipped…` row and the `battencli` `wire_seams_test.go` update |
| D4 a verified send leaves its text in the sent-texts file | `TestRun_Send_RecordsTheSentText`. It checks the content, but not the ordering "before typing". |
| D4 a failed append does not block the send | **None** |
| D4 a send with no run directory records nothing | **None** (trivial) |
| D5 the prompt is the instruction; a `<task-notification>` or slash command yields none | The two new `TestParseSessionSignals` rows, plus the captured-session golden rows |
| D6 predecessor selection: stopped, waiterless-running, done, live waiter, other role, empty role | `TestStart_CarriesOverOperatorInstructions` rows |
| D6 first turn start skipped, prompt offset respected, sent texts removed, repeats dropped, predecessor's own carried instructions merged | The same table, via `predecessorEvents` |
| D6 prompt block appended, carried instructions stored in the new record, role key stored, one notice quoting the instruction | The same table |
| D6 `died` and `timeout` predecessors count as unfinished | **None** |
| D6 newest run of the key wins (`CreatedAt` order, tie-break by directory name) | **None**. The table seeds one predecessor only. |
| D6 failure paths: missing root directory, unreadable events file, engine without `SessionSignalParser`, unregistered engine, malformed sent-texts line | **None** |
| D6 notifier nil, a notifier error only logged, instruction truncated at `noticeInstructionMax` | **None** |
| D6 carry-over is read before the orphan sweep | **None** |
| D6 carry-over across `StartRetries` attempts: carried block kept, one notice only after success | **None** |
| D6 the Warn log line | **None** |
| New `RunState` fields optional, so an older record reads | **None** (`omitempty` only) |
| D7 and D8 no subagent hook and no auto-continue key | Pinned only implicitly: the golden compares whole documents, so an added key would fail it |

**Findings**

1. MEDIUM, design. `internal/shuttleengine/carryover.go:116-121` (`unfinished`) and `:132-146` (newest-run selection). `died`/`timeout` are never tested, and no test seeds two runs of one role key, so newest selection and the tie-break are unpinned. Two more rows in `TestStart_CarriesOverOperatorInstructions` would cover both: `OutcomeDied`/`OutcomeTimeout` predecessors, and an older unfinished run beside a newer done one, which must carry nothing. Evidence: `carryover_test.go` table, one `run-old` only.

2. LOW, design. `internal/shuttleengine/carryover.go:49-55` and `run.go` `typeVerified`. A failed sent-text append not blocking the send (D4) has no row. One row would cover it: a run directory that cannot be written to, where `Send` still succeeds.

3. LOW, design. `internal/shuttleengine/carryover.go:67-70`, the `operatorInstructions` failure paths (events file unreadable, engine without the parser) and `carriedInstructions`' missing-root and unregistered-engine paths. All return none, untested. One row in the same table would cover them.

4. LOW, design. `internal/shuttleengine/carryover.go:170-181` (`noticeCarriedInstructions`). Truncation at 200 runes, a nil notifier and a notifier error have no row. The focus asks to check "notifier line is sent once and also logged at Warn". The once part is pinned by `wantNotices`; the Warn log is not.

5. LOW, design. `internal/shuttleengine/run.go:549-562` (`start`). No test pins that the carry-over is read before `sweepOrphansOpportunistic`, or that a `StartRetries` re-launch keeps the carried block and notifies once. The table runs a single launch.

6. NIT, consistency. The old-record-reads-clean constraint (`PATTERN-tolerant-records`) has no row decoding a `run.json` without `roleKey`/`carriedInstructions`. Existing record-reading rows only partly cover it.

Test economy: no duplicate or per-helper test found. The `Prepare` and `CreelMailHook` changes extend existing tests.

**Tool calls:** none. The manifest already held every test diff.
