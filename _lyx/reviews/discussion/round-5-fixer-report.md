# Fixer report — discussion round 5

All six findings were validated against the code and fixed.

## Fixed

- **F1** (events-file mtime moves batten's activity): confirmed in `internal/shuttleengine/activity.go`, where `LastActivity` takes the newer of the transcript and events-file mtimes; a grep found no other production reader that stats the events file.
  D1's Bound now names that second input and bounds it: only the `Notification` `idle_prompt` append lands while the transcript is still, so batten's quiet notice (45-minute window) fires later by at most Claude Code's idle threshold per idle notice, accepted in shadow mode.
  Added an Open risk (repeated `idle_prompt` within one idle stretch is unverified) and an acceptance row pinning the `LastActivity` move.
- **F2** (stamp and payload split across polls): confirmed, `readEventsFrom` in `internal/shuttleengine/wait.go` consumes every complete line, so a stamp line alone is consumed.
  D1 now requires an incremental reader to carry unpaired stamp lines into its next batch; the acceptance row gains the split-batch case.
- **F3** (transcript-fallback waiting turn ends): confirmed in `claudeengine/background.go`, where `outstandingBackgroundTasks` falls back to `transcriptHasUnmatchedLaunch` over the transcript at parse time.
  D2 names the fallback as a parse-time read; D3 limits the signal-only list to payload-reported waiting turn ends and adds the fallback turn end to the per-reader list; an acceptance row covers a late reading.
  The matching rejected-alternative entry in `support-log.md` names it too.
- **F4** (precedence against output-files-first): the D3 `idle-done` sentence and the `PATTERN-completion-signal` Constraint now cover only the output-derived negative states, say `dead` and `unknown` rank above them by design because no consumer acts on the state, and require the first consumer acting on `dead` to apply the output check.
- **F5** (turn start entries per reader): `busy` from a turn start now has cause `turn`, a repeated turn start records no entry, the newest turn start's entry is listed as per reader, and the history paragraph says record-on-change makes the entry after a per-reader entry per reader too; the reducer acceptance row states the cause and the coalescing.
- **F6** (chunking measurement in D1's Rationale): completeness check done, the requirement stays in the Decision and the acceptance row; the measurement sentences are cut down to one line saying the stamp is a separate short append so the payload's write stays today's.

## Disputed

None.

## Deferred

None.
