---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/creelengine/deliver.go:429
    summary: the reserved-by-a-read path of carried mail (snapshot split and markNoticeRead guard) has no test, though the summary claims a reserved message is never journaled read twice
    fix: add a row to TestScheduler_Reminders in internal/creelengine/deliver_test.go that posts a program notice, calls Store.Read without acking before the worker probes, asserts no notice line is typed and the message is reminded, then Ack and assert the log holds posted and read once each.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: consistency
    location: internal/creelcli/cli.go:296
    summary: the post help still says the receiver gets a one-line reminder and reads the message with lyx creel read, which no longer holds for EXTERNAL and program mail
    fix: in postCmd's Long text, say that a message from an agent strand is announced by a reminder and read with lyx creel read, while mail from EXTERNAL or a program is carried in full in the typed line and needs no read.
    origin: lens:code-affected-set
  - id: F3
    severity: LOW
    class: consistency
    location: docs/overview.md:436
    summary: the creel module entry still describes creel as mail an agent pulls with a reminder, and leaves out that non-agent mail is carried in full in the typed line
    fix: in the creel bullet of docs/overview.md, add that mail from any sender but an agent strand is carried in full in the line creel types, and only agent mail is announced by a reminder and read with lyx creel read.
    origin: lens:code-affected-set
  - id: F4
    severity: LOW
    class: design
    location: internal/creelengine/deliver.go:594
    summary: a notice typed while a read holds it reserved is left unread by markNoticeRead, so if that read is released rather than acked the line is typed a second time
    fix: either accept and record it in the markNoticeRead comment as a bounded retype, or have snapshot skip a reserved notice entirely, which it already does for typing, and leave the doc as is; the smaller change is the comment on markNoticeRead naming the retype after a released read.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: design
    location: internal/creelengine/deliver_test.go:375
    summary: no row pins that a non-owner's notice is not typed under a hold, or that a due inject is typed before a notice
    fix: extend the existing hold row in TestScheduler_Reminders with a program notice from a sender other than the hold owner that must stay untyped, and the first notice row with a queued inject that must be typed before the notice.
    origin: lens:code-tests
---

### [MEDIUM] Reserved-by-a-read path has no test

**Location:** internal/creelengine/deliver.go:429 and :594
**Issue:** The summary names the case where a concurrent read holds a notice reserved as a design decision.
Neither the snapshot's `s.reserved[id] == ""` branch nor the guard in `markNoticeRead` is exercised by any row; the only `Release` use in `deliver_test.go` is in the sweep test.
A regression would type or journal a notice a read already holds, and no test would fail.

### [LOW] post help is stale for carried mail

**Location:** internal/creelcli/cli.go:296
**Issue:** The help text says the receiver gets a reminder and reads the message with `lyx creel read`.
That holds only for agent mail now.

### [LOW] docs/overview.md creel entry omits carried mail

**Location:** docs/overview.md:436
**Issue:** The project rules update `docs/overview.md` when observable behavior changes; the creel bullet still describes only the reminder-and-pull model.

### [LOW] A released read lets a typed notice be typed again

**Location:** internal/creelengine/deliver.go:594
**Issue:** `markNoticeRead` leaves a reserved message to its read.
If the client dies before the ack and the read is released, the message is unread and unreserved, so the next probe types its line again.
The journal never records two reads, so the summary's claim holds; the cost is one repeated pane line, rare and bounded.

### [LOW] Hold-owner filter and inject-before-notice ordering unpinned

**Location:** internal/creelengine/deliver_test.go:375
**Issue:** The hold row has no non-owner notice, and no row queues an inject beside a notice, so the `snapshot` hold filter for notices and the inject-first order have no row that would fail if broken.

## Rejected

- Replacing the `lyx creel read` substring with `[creel]` in `watcher_test.go`: an existing assertion was edited, not a new one added on stencil prose, so test economy does not apply.
- `StrandMailNamesTheIdAndTheReplyCommand` in `TestWriteMessagesText` overlapping the CLI text row: the unit table also covers EXTERNAL, program, reply and returned headers, so most of it is unique.
- The flag name `--envelope` where the decision record says `--json`: the board entry asks only for "JSON only behind a flag", and the summary gives a reason that `TestOwnJSONFlags` backs.
