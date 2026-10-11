---
round: 1
focus:
  - "Provider seam: check that Claude hook names, payload fields, source strings, notification types and the stray-character judgment appear only in `claudeengine`, and that `shuttleengine` sees only `SessionStartSource`, signal kinds and box readings."
  - "No-capability fallback: for an engine without `SessionSignalParser`, `SessionProber`, `StrayInputReader` or a session-start-recording launch, settle whether every path cards 3, 4, 6, 7, 8 and 10 touch behaves as before."
  - "Card 7 gating: settle whether the plan's condition for building `startup-needs-session-start` (card 1's corpus shows `SessionStart(startup)` landing on its own) is met by the captured corpus, and whether the diff matches that outcome."
  - "Confirmation offsets: in card 6, settle whether `/clear` and `/compact` are confirmed only by a matching `SessionStart` line past the pre-send offset, including a stale line before the offset and a mismatched source."
  - "Stray-character clear: in card 10, settle whether the clear happens at most once, only behind the full guard reading (idle, empty-but-for-one-character box, no draft), and that the character is read as residue afterwards rather than as operator input."
  - "Fork transcript tail: in card 5, settle whether the pending call comes from the newest tool-calling message only, and how a tail cut mid-line or a transcript with no tool-calling message in the tail is handled."
  - "Dead-session end: in card 8, settle whether the registry or a `SessionEnd` line alone ends a run as died or done, and whether a live process can be misread as gone."
  - "Per-card mechanical checks and test economy: for each card's groups, confirm the type-specific check ran against every group's own targets; for new top-level tests, check duplication against existing table tests and that `cmd/lyx/testdata/test-budget.yaml` is edited where a package passes its budget row."
---

Focus is set from the plan's index and shared decisions only; no diff was read for this seed.
