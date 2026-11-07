---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/slackengine/outbound_test.go:260
    summary: The stale-mail count line has no test under a failing outbox write, though the diff removed the rule that held it back
    fix: In TestOutboundRead/OutboxWriteFailingAfterAnAcceptedPostAcknowledgesTheMailAndNeverRepostsIt, add a row whose batch holds mail older than a day (Posted 25 hours before the clock) plus fresh mail, with the same onPost that turns the outbox path into a directory. Across two reads with the first acknowledgement refused, assert the count line (`2 messages ... lym creel log --to ly:SLACK`) is in the posted texts exactly once.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/slackengine/outbound.go:93
    summary: No test pins saveFault riding with a failed post or a rate limit, the errReadBudget return, or unsaved clearing after a later good write
    fix: Add rows to the same outbound write-failure test. One breaks the outbox through onPost and makes a second message's post fail, and asserts the read error carries both the post failure and `outbox file`. One removes the directory at the outbox path before the next read and asserts that read returns nil and the file holds the record.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:870
    summary: The overview still states the old outbound guarantee, never twice for one acknowledged read, which doc.go and the outbound.go header replaced
    fix: In docs/overview.md line 870, replace "at least once and never twice for one acknowledged read" with "at least once, and a message Slack accepted is not posted again".
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: internal/slackengine/doc.go:44
    summary: The package doc and the outbound.go header promise an accepted message is never posted again, but a relay restart while a record is held only in memory can repost it
    fix: In doc.go, add one sentence after the sentence on the in-memory record saying the record is lost when the relay process ends, so a restart before a good write can post that mail again. Make the outbound.go header say "never again within one relay process once Slack accepted it".
---

### [MEDIUM] Stale-mail line untested under a failing outbox write

**Location:** internal/slackengine/outbound_test.go:260
**Issue:** The diff deletes `errOutboxWrite` and the rule that the stale-mail line waits after a failed write. Its replacement is that the in-memory record keeps `Stale` at 0 once the line posts. Both rows of the new test send one fresh message and no old mail, so `Stale` is never above 0 and a regression that reposts the line passes.

### [LOW] saveFault, errReadBudget and unsaved-reset paths untested

**Location:** internal/slackengine/outbound.go:93
**Issue:** No test combines a failing write with a failed post, a rate limit or the posting budget, so the `errors.Join` at line 97 and the `errReadBudget` return are unexercised. No test restores the outbox file, so `unsaved` clearing at line 266 never runs.

### [LOW] docs/overview.md carries the old outbound guarantee

**Location:** docs/overview.md:870
**Issue:** The line says "at least once and never twice for one acknowledged read". The diff reworded the guarantee in `doc.go` and the `outbound.go` header, so the overview now says something different from the package it describes.

### [LOW] Guarantee stated without the restart bound

**Location:** internal/slackengine/doc.go:44
**Issue:** The landing summary says a relay that dies after a failed write and before the next good one can still repost. `doc.go` says "a message Slack accepted is not posted again", and the header says "never again once Slack accepted it", with no such limit.

## Rejected

- Lens code-correctness reported no findings, and the handler's own run of the code agreed.
- A lens note that `read` returns an earlier `saveFault` after a final `save(outboxRecord{})` succeeds: state is consistent and the log line reports a real fault.
- A lens note that `decodeAnswer` has no test on an unreadable answer: the error is unchanged and was untested before.
- A lens note that `cloneOutbox` has no test: a per-helper test would duplicate coverage the public surface gives.
- Scope against the decision record: all three wants are met. Each of the five methods has a documented-body test, an accepted post is not reposted after a failed write, and the summary names #1023 with two ways to clear it.
- Summary claims: the `outbox.json` file name matches `OutboxFileName`, the hub has no outbox file, and #1023 shows as posted and unacknowledged in `lym creel log --to ly:SLACK`. The day-old rule matches `staleMailAge` for mail posted 2026-11-07T17:38:40Z.
- PATTERN and commit-message checks: the rubric declares no PATTERN changes, no `Pattern-Departure:` trailer appears, and the daemon-wakeups backoff text is unaffected.
