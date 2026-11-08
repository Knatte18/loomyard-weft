---
round: 1
focus:
  - "D1 and D2, inbox hint files: check whether the hint's file name, which carries a channel or channel and thread id, is validated before it names a read, and what a planted or malformed name can make a relay read or write. Settle whether D1's Bound covers it."
  - "D1, lock and ack: check the Decision states what happens to a holder that stalls while holding the lock without ending (a hung connection, a blocked hint write), and that the acknowledgement rule has a bound where Slack's docs name no ack deadline."
  - "D3 against the board note `slack-login-prompt` (running): check whether D3's one-token login prompt, which defers to 'whatever way `slack-login-prompt` lands', contradicts or duplicates that entry, or is a parent-level question. Read the entry and file it as `decision` only if the writer cannot settle it by citing or dropping."
  - "D3, D5 and D6, user token: check each use of the user token (unarchive, invite) and each adopt or reopen path against the stated bound, including a channel adopted on `name_taken` and the 'held by no other orch's entry in any live hub's map' test when a hub's map is unreadable or stale."
  - "D5 and Open risks: decide whether the archived-name risk is a deferred research question (a fact the design rests on, left for the first live reopen) or a fact the design handles both ways. Say which, with the evidence."
  - "D6 and D8, refusal and reopen loops: check that `is_archived` and `channel_not_found` re-posts are bounded, and that the 'refusal cap' D8 names is defined in the record or a cited existing rule."
  - "D7 inbound filter: check that dropping the `@<slug>` prefix and admitting `file_share` and `thread_broadcast` leaves the operator-only and bot-skip guards intact for every channel, and that no guard is weakened without a stated bound."
  - "Acceptance criteria against Decisions: check each Decision's acceptance bullets are behaviour a test can check, `Review:` marks are on prose-only criteria, and D11's seam and D2's cursor rules each have a bullet that would fail if the rule broke."
  - "Relocation pass: check Rationale, Auto-mode assumptions and Constraints for deliberation or restated Decisions that belong in `support-log.md`. Before any move, extract any requirement the plan writer needs into Decisions or Constraints."
---

Seed focus for round 1 of the slack-v2 discussion pair.
Each entry names a place to look and a question to settle; none states a verdict.
Checked against the rubric: none asks for a missing "Notes for the plan writer", a rejected alternative in the record, or a call-site enumeration.
