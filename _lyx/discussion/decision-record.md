# Decision record — creel-compact-notices

## Goal

A program's notice reaches its receiver in the line creel types, with no read, and `lyx creel read` prints plain text a receiver acts on directly.

## Scope

- `internal/creelengine`: the delivery loop types a program sender's one-line message as its own line and journals it read; the reminder lists only the rest of the unread mail; `Read` returns every unread message by default.
- `internal/creelcli`: `lyx creel read` prints plain text, with `--json` for the current envelope; `--all` goes.
- The Go client's read, the orch role stencil's mail steps, the sandbox creel suite and the package docs follow.
- Out of scope: how batten or any other program words its notices, and notices posted under a strand claim, which stay agent mail.

## Decisions

### A notice is a program's one-line message

Decision: a notice is an unread message whose original sender is a program mailbox name (upper-case, never EXTERNAL), whose body is one line without control characters and at most 2048 bytes, and that no read holds reserved.
Rationale: only a program sender takes no reply, a program name is accepted from an orch pane or the Go client only, so its text cannot be an agent posing as an instruction, and one line is all a pane line can carry.
Bound: a program message that is multi-line or longer stays mail announced by the contentless reminder; EXTERNAL and strand senders are never notices.

### The notice line is typed after injects and before the reminder, one per probe

Decision: a probe types the first due inject, else the oldest notice as `[creel] <sender>: <body>`, else the reminder; under a hold only the hold owner's notices are typed.
A typed notice, or one whose landing was not confirmed, is journaled `read` by the receiver once the send returns; a busy send leaves it unread for the next probe.
Rationale: one delivery per probe keeps the existing probe and backoff loop, and the read record keeps the journal and `lyx creel log` the full history.
Bound: a daemon crash between the send and the read record types that notice once more at the next start; no other path retypes one.

### The reminder lists only the mail that is not a notice

Decision: the reminder's count, listed ids and backoff set cover the unread messages that are not notices, so a receiver whose unread mail is all notices gets no reminder.
Rationale: a notice already carried its content and is marked read when typed.

### Read returns every unread message

Decision: `Read` with no sender filter returns every unread message oldest first, `--from` narrows it to one sender, and the `--all` flag and the request's `all` field go.
Rationale: the receiver handles all its mail in one read; a reader that sends the old field still reads all, since unknown keys are ignored.

### Plain-text read output

Decision: `lyx creel read` prints each message as a header line and then its body, messages separated by a blank line, and `nothing unread` when there is none.
A message from a strand or EXTERNAL opens with `#<id> from <sender>` and ends the header with how to reply, `lyx creel post --reply <id> <text>`; a program's message opens with `from <sender>` and carries no id or reply hint.
The header names the message it answers for a reply and the receiver that ended without reading it for a returned message, and adds `posted <RFC 3339 time>` only when the message is more than five minutes old by the CLI's clock.
Envelope, kind and path are left out; `lyx creel log` keeps them.
`--json` prints the current JSON envelope unchanged.
Rationale: the operator decision of 2026-10-18; a program's sender stays in the header because the body alone does not say which program wrote it.

### Added after Discussion (parent, 2026-10-18): Which mail needs creel read

Decision: Only mail from an agent strand keeps the contentless reminder and needs lyx creel read. Mail from every other sender, program senders and ly:EXTERNAL alike, is carried in full in the reminder line and marked read on delivery. Every carried line starts with its message id, so a program that takes replies, such as the orch watcher with its written/defer answer, can be answered with lyx creel post --reply <id> without a read.

Rationale: ly:EXTERNAL has no strand, so a reply to it is undelivered and its mail needs no read. Case does not tell whether a sender takes replies (the watcher does, batten does not); always showing the id costs a few characters and keeps creel from knowing which programs answer.

## Constraints

- `PATTERN-creel-store`: the journal is written only by the daemon in `internal/creelengine`, and panes are reached only through the `Delivery` seam.
- `PATTERN-tolerant-records`: no new record field is needed; the notice's read record is the existing `read` event.
- `PATTERN-test-speed`: delivery tests run on the fake clock and the fake seam.
- Docs land in the same commit: `internal/creelengine/doc.go`, `internal/creelcli/doc.go`, the orch role stencil and the sandbox creel suite.

## Auto-mode assumptions

- The 2048-byte cap fits every batten notice, whose error and path parts are capped at 200 and 400 characters.
- The five-minute age threshold is "a few minutes" in the operator decision.
- Refusals of `lyx creel read` keep the shared JSON error envelope every creel verb prints.

## Open risks

- A receiver that relied on the `[batten notice]` mail being readable after delivery now finds it only in `lyx creel log`; the orch role stencil names that place.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - reserved-by-read-path-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- A program's one-line message to a live idle strand is typed as `[creel] <sender>: <body>`, is read afterwards, and no reminder follows for it.
- Strand mail beside a notice is reminded with a count and list that leave the notice out.
- A multi-line or over-cap program message, and EXTERNAL and strand mail, are reminded as before.
- A busy send leaves the notice unread and it is typed at the next idle probe; a not-landed send marks it read.
- Under a hold, only the owner's notices are typed.
- Plain `lyx creel read` returns every unread message from every sender in one call, and `--from` narrows it.
- `lyx creel read` prints plain text: id, sender and the reply command for strand and EXTERNAL mail, sender only for program mail, the time only past five minutes, and `nothing unread` for none; `--json` prints the JSON envelope.
- A read whose output cannot be written still leaves the mail unread.
