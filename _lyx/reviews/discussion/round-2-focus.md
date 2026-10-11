---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, D5 (foreign-box error after the paste-submit) and D7 (watcher retry) and D8/D9 (batten resume retry): settle whether every outcome where the sent text may still sit in the operator's input box is kept apart from outcomes where nothing landed, so no caller re-types the same line. `resumeParkedDriver` in `internal/loomcli/driverresume.go` refuses to retry a send that may have delivered."
  - "D6 puts the shuttle start prompt and skill-load message behind D3's operator-activity hold and D5's foreign-box failure. `loadSkillsThenPrompt` in `internal/shuttleengine/run.go` turns skill-load or prompt failures other than `ErrSubmissionNotLanded` into a died start, and `lyx shuttle send` returns the error without retry. Settle whether the record states what a held or foreign-box start delivery does."
  - "D5 accepts a box holding exactly the collapsed-paste placeholder before sending a guarded Enter. Settle whether the Decision states what can now be submitted and what bounds it, given D3's activity check runs once before the write."
  - "D7 removes the foreign-box and busy failures from the consecutive-tick-error count. `deliverNotice` in `internal/orchengine/watcher.go` runs in the idle phase, gated by the watcher's own idle probe. Settle whether a notice that is held repeatedly is bounded or surfaced, outside the cycle phases."
---

Questions only; severity and verdict are the reviewer's.
