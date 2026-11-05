---
round: 1
focus:
  - "decision-record.md D4, D5, D6, D7: the relay turns Slack text into operator-authority mail and can set presence, route to any beam's orch, and post creel mail outward. For each edge (the `@<slug>` route, the `away`/`back`/`status` texts, outbound posting by stamped sender, the `SLACK` program claim), check that the Decision states what it can let through and what bounds it, and that the bounds agree across D2, D4, D5 and D7."
  - "decision-record.md D2, D4, D5 and Constraints: list every fact the design rests on about Slack (scopes per channel type, `conversations.history` cursor and ordering semantics, whether the operator's own phone messages can carry a `subtype`, how `Retry-After` is delivered, `chat.postMessage` behaviour for a bot that is not a member) and about creel (the 4 KiB cap, how a program becomes a known receiver, the 7-day retention, read/ack/release semantics). Check each is either answered with a named source or version, or is a choice rather than a fact; flag any left to Plan or an implementer to find out."
  - "decision-record.md D3, D6, D8 against the code they name: confirm the premises about `runproc` roles and `EndRole`, the prime dobby's passes, `loomshed.OperatorWaitRows`, and the status history entry's `At` field are what the record says they are today, and that no Decision depends on a premise the exploration did not establish."
  - "decision-record.md Open risks, Auto-mode assumptions and Constraints: check each line carries something the plan writer needs, that no Open risk restates a D-bound, and that no Open risk is a question the record could have settled itself or a `decision`-class question for the parent. Check the `pr-review-wait-silent` line does not ask the run to edit a board entry."
  - "decision-record.md Acceptance criteria against Decisions: check every D-id has a behavior criterion a test can check, that each `Review:` criterion is truly prose-only, and that D3's dobby-restart, D6's resume-pointer line and D8's restart and re-entry bounds are covered where the Decision states a behavior."
  - "support-log.md against decision-record.md: check no requirement or constraint lives only in the support log (the plan writer never reads it), for example the Socket Mode reasoning or the lens-review fixes, and that the record's Decisions carry no rejected-alternative narrative that belongs in the log."
---

Seed focus for round 1.
Entries name where to look and which question to settle; none states a verdict.
The three `Do not flag` items and the writer's stencil bounds (no exact signatures, no exhaustive pattern research) were checked against each entry.
