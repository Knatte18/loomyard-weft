# Fixer report — discussion round 1

Every finding was checked against the cited code (`internal/loomcli/driverresume.go`, `internal/shuttleengine/run.go`, `internal/shuttleengine/readiness.go`, `internal/shuttleengine/claudeengine/session.go`, `internal/orchengine/watcher.go`, `internal/shuttlecli/send.go`, `internal/seatengine/notice.go`), and every premise holds.
All eleven are fixed in `_lyx/discussion/decision-record.md`, with the new self-picks recorded in `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` passes.

## Fixed

- **F1 (BLOCKING):** D5 now splits the foreign-box outcome.
  A box without the sent text is a plain hold.
  A box holding the sent text beside other text fails with its own sentinel (`ErrSentTextInDraft`), and shuttle records the pending send's text and events offset in the strand's run state.
  The retry of the same text, once the box reads empty, returns landed without typing if a turn start past the offset exists, and types once otherwise.
  D8 states that a resume line sent inside the draft resumes the driver, whose re-step removes the park marker, so batten takes no resume branch again.
  The leftover-text outcome and the record's misread case are R6.
- **F2 (MEDIUM):** D7 covers shuttle `Start` and `lyx shuttle send`.
  A held start delivery stays a died start after `send_ready_timeout_s`, stated as a bound with R7 and an acceptance criterion.
  `lyx shuttle send` reports the hold and leaves the retry to its caller, deduplicated by D5's record.
  The rationale no longer claims every caller retries.
- **F3 (MEDIUM):** D5 gains a Bound: placeholder acceptance can submit an operator's own paste.
  It is limited to a box that read empty before the write, to the send's `submit_confirm_timeout_s` window, and to a fresh operator-activity check before each guarded Enter.
  R3 now points at that bound, and the acceptance criterion covers the Enter held by activity.
- **F4 (MEDIUM):** D7's Bound adds that a held step outside a cycle, such as an idle-phase notice, records its hold reason in `State.Stuck` as a probe hold does and is cleared by a passing send, with a matching acceptance criterion.
- **F5 (LOW):** D5 names the no-readable-box outcome after the paste-submit: no Enter, keep reading within the window, and fail as `ErrSubmissionNotLanded` with no Enter and no clear key at the window's end.
- **F6 (LOW):** Both foreign-box failures wrap `ErrSessionBusy`, not `ErrSubmissionNotLanded`.
  D7 drops the "plan picks which" choice, and D8 states that a never-appeared send keeps its non-retryable refusal.
- **F7 (LOW):** D2 states that `-p` brackets the paste only when the application enabled bracketed-paste mode.
  New assumption A7 covers it, and the `llm`-tier criterion also checks that a long paste collapses into the placeholder.
- **F8 (LOW):** D3's empty-box condition states that a missing box fails it on both paths for a box-reading engine: through D4 on a known reading and through the engine's idle reading on the unknown fallback.
- **F9 (LOW):** New acceptance criterion for the activity-query fail-open; the `driver_busy` criterion names `lyx loom resume` too.
- **F10 (LOW):** The spawn-observability constraint names both new tmux invocations, the activity query per reading and the paste-submit per write, and says both are logged.
- **F11 (NIT):** D9 states that the orch notice names the slug, stall and remedy and never quotes the draft, and the batten criterion says so too.

Support log: two rejected alternatives (a non-retried in-draft error, an unbounded `Start` wait) and ledger entries Q13 to Q15.
The `## Review rounds` section is untouched, since its entries belong to the gate.

## Disputed

None.

## Deferred

None.
