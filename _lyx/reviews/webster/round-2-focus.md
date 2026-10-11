---
round: 2
focus:
  - "In internal/shuttleengine/send.go, settle whether an engine that does not implement SessionSignalParser confirms a /clear or /compact paste on the empty box at once, as before the session-start confirmation was added, and whether an engine that parses signals but records no session-start line still waits out the submit window."
  - "In internal/shuttleengine/session_test.go, settle whether a table row seeds the events file with a session-start line of the landing source before the pre-send offset, with no line written on the paste, and expects no confirmation before the window closes."
  - "In internal/shuttleengine/claudeengine/activity_test.go, settle whether a test exercises holdsPendingToolCall on a transcript larger than the initial read chunk, with a pending call in the first entry of a parallel batch that lies before the first window's start, so the window widens."
---

The entries ask where the round-1 fixes landed and whether they hold; each is a question to settle, not a verdict.
