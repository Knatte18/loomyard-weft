---
round: 1
focus:
  - "Attack surface in D2/D3/D8: the hand-written `deliver` and `confirm` requests are new routing edges any process of the user can send. Check that each Decision states what the request can skip or let through and what bounds it, including a `confirm` probe used to learn which outbound messages a hub holds, and a `deliver` flood against a receiving daemon."
  - "Guard bypass in D3/D5: D3 admits that journal, record, socket, orch-pane and reed routes defeat the sender check. Check that D5's trust grant states what a forged or injected trusted orch can reach (the orch's own authority only), and that D1's record being writable is bounded where the lookup is used for trust, not only for addressing."
  - "Deferred facts in D1-D5 and Constraints: look for any fact the design rests on that is left to Plan or an implementer, such as how `agentname.Parse` treats a foreign shortname, how retention or the sweep treats the new outbound event, or whether `SO_PEERCRED` sender identity is available at the confirm path. Each needs its answer recorded with the evidence, not a hand-off."
  - "Open risks: the first line asks the parent to confirm a stricter reading of the entry; the second raises a conflict with `sandbox-closed-area-build`. Settle whether each is a `decision` finding, a choice the writer already made (so the line should drop), or correctly raised. Check both against any `Decision for creel-cross-hub-mail:` board note and the entry body."
  - "Board edits in D9: the note is filed by a card, not a task, and the entry body is untouched. Confirm no Decision changes this task's own entry or files or promotes a task."
  - "Relocation and leanness: look for deliberation narrative in `decision-record.md` that the plan writer does not need (rationale restating rejected options, the entry-versus-design departure in D1 and Open risks) and for requirements living only in `support-log.md`, which Plan-Write never reads. Extract any requirement into Decisions or Constraints before proposing a move."
  - "Consistency across D2, D4 and the acceptance criteria: each D id must have a behavior-checkable bullet, the retry and one-hour return rule in D2 must agree with the D4 bounds on late replies and retention, and the Auto-mode assumptions must match the Decisions they cite."
---

Round 1 seed from a first read of both files.
The writer's stencil and the discussion spec bar exact signatures, `file:line` citations and exhaustive call-site research, so no entry asks for them.
A missing rejected alternatives list and a missing plan-writer notes subsection are not flagged.
