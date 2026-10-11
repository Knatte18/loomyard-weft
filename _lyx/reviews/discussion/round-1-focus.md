---
round: 1
focus:
  - "Attack surface, D3 and D4: each removed or downgraded guard (fail-open on an activity-query error, unknown-readiness falling back to the pane reading, box-less engines never held on an unreadable box) states what it lets through and what bounds it. Settle whether each bound is stated in the Decision itself."
  - "Attack surface, D5 and D6: the collapsed-paste placeholder is accepted as 'ours' and the leading Escape is dropped. Settle whether the Decisions state what an operator's own paste or an open menu can now get through, and what bounds it, beyond what Open risks mentions."
  - "Attack surface, D8 and D9: the new `driver_busy` refusal, its retry without a deadline and the once-per-stall notice. Settle whether the Decisions bound how long and how often the retry runs and what ends the stall."
  - "Consistency, D5 against D7 and D8: the foreign-box error is wrapped as `ErrSubmissionNotLanded` or `ErrSessionBusy`, the watcher treats it as nothing typed, and loom start maps it to `driver_busy`. Settle whether the three Decisions agree on which callers see which error and whether any caller still counts it as a failure."
  - "Consistency, Scope against Decisions and Acceptance criteria: every item In Scope is covered by a Decision and by an acceptance criterion, and no Decision touches something listed Out. Settle any item that appears in only one of the three."
  - "Relocation: Decisions carry Decision, Bound and Rationale only. Settle whether any Decision or Open risk holds deliberation narrative that belongs in the support log, after checking it carries no requirement the plan writer needs; extract any such requirement into Decisions or Constraints first."
  - "Design, D1 and D2: the atomic paste-submit, the named buffer, the pane resolution and the claim that the merge window shrinks to the gap before one tmux invocation. Settle whether any premise about tmux or Claude Code behaviour is stated as fact where it is unverified, and whether Auto-mode assumptions or Open risks name it."
---

Focus is on guard-bound statements, cross-Decision consistency and relocation, the rubric's own checks.
Missing rejected alternatives in the decision record, the optional plan-writer notes and call-site enumeration are by design and are not focus.
