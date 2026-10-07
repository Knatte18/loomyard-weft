---
round: 1
focus:
  - "D4 and D5 (hold replaces the asking stop): settle whether the bound on a held run is complete, i.e. what a held run can now let through (an unanswered or unnoticed agent, a notice dropped by `QueueNotice`) and whether the deadline, liveness check and single notice are stated as the actual limits."
  - "D2 (payload `background_tasks` authoritative, transcript skipped when the key is present): settle whether the stated bound covers a payload that under-reports, and whether the acceptance criteria and live scenarios tie to that bound."
  - "D6 (attach of legacy `asking` run.json): settle whether attaching from `AskingOffset` can re-notify, double-attach or resume a stale strand, and whether the stated bound closes that."
  - "D8 and D9 (tail appended by `lyx shuttle send`, expired shell counted as a hold): settle whether each states what it can now skip or let through and what bounds it, including a sender that already ends with the tail."
  - "Consistency across Goal, Scope, Decisions, Constraints, Auto-mode assumptions and Acceptance criteria: settle whether every in-scope item (retired `OutcomeAsking`/`AwaitOperator`, `submit_redraw_settle_ms`, corpus, live scenarios) has a Decision and an acceptance criterion, and whether any Decision contradicts another (e.g. D4's deadline bound against the 30-minute assumption)."
  - "Completeness-before-leanness: settle whether `support-log.md` holds any requirement or constraint the plan writer needs that `decision-record.md` lacks, and whether `decision-record.md` carries deliberation narrative that belongs in the support log."
---

Focus applies the rubric's `Do not flag` list and the discussion stencil's no-signatures, no-`file:line`, no-exhaustive-pattern-research limits: none of the entries asks for those.
