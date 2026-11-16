---
round: 1
focus:
  - "D8 and D12: each runs or installs something new (plugin SessionStart command hooks at Prepare; Codex PreToolUse and PostToolUse hooks). Settle whether each Decision states what it can run or let through and what bounds it, and whether D12's claim that a PostToolUse `reason` reaches the model rests on recorded evidence or is left to be found out later."
  - "D5, D6, D9: the Codex config keys they set (`developer_instructions`, `default_subagent_reasoning_effort`, `max_concurrent_threads_per_session`, `project_doc_max_bytes`). Settle whether each key's existence and semantics rest on a cited fact, or a fact the design needs is deferred to Plan or an implementer."
  - "D4 and D2: the escape hatches (a hand-written prompt marker, an allowlist entry for a Go literal). Settle whether each bound is concrete enough to hold, and whether D4's guard against the orch launching on Codex has a stated bound."
  - "D3: removing the shared edit directive stencil and the exemption in the edit-rule guard. Settle whether the guard is downgraded or bypassable and whether the Decision states the bound."
  - "Open risks and Q14: settle whether the open risk about `codex-assist-review-trial` is a question only the parent can settle (class `decision`) or one the writer could settle, and whether any Decision or criterion asks a run to file or promote a task, or edit this task's own board entry (D11 edits other entries)."
  - "Acceptance criteria: settle whether each criterion is a checkable behavior or a `Review:` line, and whether any Decision's claim (D2 Go-literal scan, D9 nested CLAUDE.md naming, D10 verb's no-write bound) has no criterion that would catch its absence."
  - "Relocation: settle whether the Rationale or Fact text in D1 to D12 carries a requirement the plan writer needs, or only deliberation that belongs in `support-log.md`, before raising any relocation finding."
---

Seed focus only; every rubric bullet still applies in full to the whole pair.
