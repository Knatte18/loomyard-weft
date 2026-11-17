---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: The fix key offered a tool_response check or a doc note, depending on whether Codex fires PostToolUse for a failed apply_patch. Nothing in the repository confirms either, so RecordEdit now skips a payload whose tool_response is an object with success false or a non-empty string or object error, with rows for failed, rejected, successful and error-false responses. The real Codex response shape stays unconfirmed.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- F1: `patchFailed` and `editHookInput.ToolResponse` in `internal/shuttleengine/codexengine/editrecord.go`; `RecordEdit` returns nil for a failed patch and its doc says so. Rows `AFailedPatchAppendsNothing`, `ARejectedPatchWithAnErrorAppendsNothing`, `ASuccessfulResponseIsRecorded`, `AnErrorFlagOfFalseIsRecorded`.
- F2: `docs/overview.md` names `apply_patch` on codex beside `Edit` and `Write`, and the Codex paragraph names the `apply_patch`-matcher edit hook.
- F3: `patchedFiles` uses `strings.TrimSpace`; row `AHeaderWithLeadingWhitespaceIsRecorded`.
- F4: `TestRecordEditAppendsAcrossCalls` calls `RecordEdit` twice on one records path and reads both hashes back in order.
- F5: rows `ABareThreadIDRolloutNamesItsThread`, `AShortRolloutNameIsTheWholeThreadID`, `ANoSessionPayloadNamesNoThread`, `ANoTranscriptPayloadNamesNoThread`.
- F6: a stamped PostToolUse line with no `tool_input.file_path` joins the `TestParseSessionSignals` input and yields no signal.
- F7: `hookedit.go` Long text and the `EditRecorder.RecordEdit` doc say each edited file.

The codexengine gate is green after the last change: `lym gate test ./internal/shuttleengine/codexengine`.

## Self-check

- fix-code-correctness, MEDIUM: the guessed `tool_response` shape is unconfirmed. Not a code change; recorded here and under F1's reason.
- fix-code-correctness, LOW: `error` of false, 0 or an empty object read as a failure. Fixed; only a non-empty string or object counts, with a row.
- fix-code-correctness, LOW: the 36-character rollout row does not pin the whole-name return. Fixed with a short-name row.
- fix-code-correctness, LOW: the no-session row does not pin the early return. Fixed with a no-transcript row, the half of the guard that matters.
- fix-proposal, LOW: `TrimSpace` also matches a hunk context line that quotes a patch header. Deferred: the reviewer's indented-header case and this one cannot both be told apart without hunk tracking, and hunk tracking would defeat F3. Both inputs are rare. The operator can choose the tradeoff.
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Gate repair

The gate refused the doc comment of `TestRecordEditAppendsAcrossCalls` in `editrecord_test.go` for a wrapped break inside a sentence.
It is now two lines, one sentence each; no other change.

## Disputed

None.

## Deferred

- fix-proposal's `TrimSpace` context-line case, for the reason under Self-check.

## Rulings

None.
