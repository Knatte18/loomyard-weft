# Card 4 — activity-from-stamps

**Edit:**
- `internal/shuttleengine/activity.go`
- `internal/shuttleengine/activity_test.go`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#SessionSignalParser`
- `internal/shuttleengine#SessionSignal`

**Intent:**
Make `ReadAgentActivity` ignore every events-file line `ParseEvents` skips, before card 5 adds the new hooks and stamp lines (the added decision "Batten activity ignores lines ParseEvents skips"), so shadow mode changes no batten notice.

Today `readRunActivity` counts the events file's mtime toward `LastActivity`.
After this card, when the engine implements `SessionSignalParser`, the events file's part of `LastActivity` is the `At` of the newest signal past the prompt offset whose `Event` is true, the newest Stop or AskUserQuestion record; a `Notification`, `UserPromptSubmit`, `StopFailure` or `SessionEnd` append, or a stamp line, never moves it.
When that newest Stop or ask line carries no time (a run whose settings predate card 5, whose only appends are Stop and ask lines), the events file's mtime counts as today, so such a run reads exactly as before.
An engine without the capability keeps today's mtime reading.
The transcript part (`TurnEndActivity`), the API-error fields and the creation-time fallback are unchanged.

This deliberately follows the added decision over the acceptance row that expected an `idle_prompt` append to move `LastActivity`: the later parent decision supersedes it, and its own acceptance row pins the opposite.

Tests, in `activity_test.go`'s existing table: a run whose events file holds a stamped Stop followed by a stamped `Notification` `idle_prompt` reads `LastActivity` at the Stop's stamp time, not the file's mtime; a run with an unstamped Stop reads the file's mtime as today; existing rows keep their expectations.
The table's engine double gains the parser through the real Claude parser's line shapes written by the test, or a double answering fixed signals, whichever the existing fake supports without importing `claudeengine` (`PATTERN-shuttle-provider-seam`).

Docs: `activity.go`'s file comment ("when each last wrote anything") and the `AgentActivity.LastActivity` field comment, which today says the last activity is the newer of the transcript's and the events file's last write, are reworded to the newest Stop or ask line's stamp, with the mtime fallback for an unstamped line or an engine without the parser; `shuttleengine/doc.go`'s activity paragraph says the same.

**ImpactSummary:** ReadAgentActivity's events-file time comes from the newest Stop or ask line's stamp instead of the file's mtime, falling back to the mtime only for unstamped lines; batten is its only caller, and the package doc and activity.go comments follow.
