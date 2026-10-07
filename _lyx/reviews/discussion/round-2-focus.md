---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, settle whether the stated bound on a held run (the Auto-mode assumptions, the Open risks and D4 Bound) matches the deadlines the autonomous callers actually set: each caller sets `Spec.Timeout` from its own config (`master_timeout_min` in `internal/websterengine/template.yaml`; `discussion_timeout_min`, `review_timeout_min` and `plan_timeout_min` in `internal/loomengine/template.yaml`), `shuttle.yaml`'s `run_timeout_min` is only the fallback, and `Attach` computes a fresh `now + Timeout` on each attach. Also settle whether the record states what a no-notifier autonomous run, including a standalone `lyx shuttle run`, now does on an asking stop."
  - "In `decision-record.md` D5, settle whether the hold notice, which types the start of the held agent's last message into the orch session, states a truncation length, a fixed notice prefix, and that the quoted text is data and not an instruction. `QueueNotice` already refuses a newline and a leading `/`."
  - "In `decision-record.md` D2 and Open risks, settle whether the transcript-fallback condition is stated the same way in both places, and whether D2 says what a present `background_tasks` key with a non-list value, and an entry with a non-`running` status, count as."
  - "In `decision-record.md` D5 and D6, settle which way a legacy `asking` record with no `AskingOffset` goes, whether the old ask is notified on attach, and whether the notified offset in `RunState` is persisted before or after the notifier call."
---

Focus targets the gating design questions and the decision gaps that share their sites.
