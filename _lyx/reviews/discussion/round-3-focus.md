---
round: 3
focus:
  - "In `_lyx/discussion/decision-record.md` D4's Bound, check the held-run deadline is stated as the spawning caller's own timeout key (with the shuttle `run_timeout_min` fallback where a caller sets none), against each caller's `Spec.Timeout` in the code (`internal/loomengine/template.yaml`, `internal/websterengine/template.yaml`, `internal/landingshed/template.yaml`, `internal/shuttleengine/template.yaml`, `internal/shuttleengine/attach.go`); settle whether any stated bound is false for a caller, and whether Open risks names an autonomous run whose spawner wires no notifier holding silently until its deadline."
  - "In D5, check the hold notice's quoted agent text: settle who owns the claim that the quote is data and not an instruction (the notice text, the orch stencil, or neither), and whether the notice's fixed prefix and delimiting mark the quote as the agent's words; check the stated handling of control characters beyond newlines and whether the length cut is on bytes or characters, against `QueueNotice` in `internal/orchengine/notice.go`."
  - "In D5 and D6, settle what a legacy asking record with no `AskingOffset` does when replay from the prompt offset in `reconstructAndWait` (`internal/shuttleengine/attach.go`) crosses several turn ends without outputs: one notice for the last held turn end or one per turn end, and whether the acceptance criteria pin that case."
  - "Check that Open risks and D2 state the transcript-fallback condition for the `background_tasks` payload identically (key absent, or value not a list), and that the acceptance criteria agree."
---

The focus targets the round-2 findings on the deadline bound, the notice text bound, the legacy replay backlog and the fallback wording.
Each entry asks a question to settle and sets no severity or verdict.
