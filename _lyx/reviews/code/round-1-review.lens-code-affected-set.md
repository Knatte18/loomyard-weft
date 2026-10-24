```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens on the `ReadAgentActivity` waiterless-run logging (activity.go, activity_test.go, doc.go).

**Per change, the set found:**

1. `ReadAgentActivity` now logs a new Warn, Debug and Info episode.
   - Return values and the live-run set are unchanged.
   - Callers are `internal/loomcli/looparm.go:159`, `internal/loomcli/decision.go:653` and `internal/battencli/wire.go:614`.
   - They use only the returned readings, so none is affected.
   - Covered.
2. New package var `waiterlessRuns` and const `waiterlessRecovered`.
   - They appear only in `activity.go` and `activity_test.go:318-339`. No other file reads or writes them.
   - A separate `EpisodeSet`, so it does not clash with `unreadableFiles`. That set is shared across `sessionread.go`, `agentwait.go`, `engineset.go` and `waitmark.go`, and keys on the same `runDir`.
   - Covered.
3. Test helpers `requireFailureEpisode`, `countLevelLines` and `singleEngineSet` exist in `fakes_test.go:84,95` and `engineset_test.go:12`. The new test uses them as defined. Covered.
4. The doc comment on `ReadAgentActivity` and the `doc.go` Agent activity paragraph (lines 427-430).
   - Both state the Warn, Debug and Info behaviour, and the record-vanished end.
   - The parent's usage-limit sentences sit at `doc.go:104-106`, in the held-turn-end section. The new paragraph is in a different section, so there is no merge damage or contradiction.
   - Covered.
5. The log message strings appear in no `.md`, `.go` or `.yaml` file outside `activity.go` and its test. No fixture or doc asserts them.
   - Covered.
6. Stencils: the diff touches none. The "Stencil wording" rule does not apply.

**Findings:** none.

Tool calls (1 of 6):
1. A grep across `internal` and the repo for the new symbols, the test helpers, the `ReadAgentActivity` callers and the message strings. My context lacked the callers outside `shuttleengine`, the helper definitions and any other consumers of the strings.
