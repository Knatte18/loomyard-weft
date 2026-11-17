---
round: 1
focus:
  - "Each Codex engine block under contracts/stencils/engine/ against its Claude copy: does it say the same thing in Codex's terms (spawn_agent with fork_turns none or all, wait_agent, apply_patch, read-only shell, developer instructions, question asked in the pane), and does any Claude-only rule get dropped or weakened?"
  - "Stencils and specs that changed (twill, webster, weaver, seat, specs): is every closed-list term now inside a Claude engine block, and does text left outside the block still read correctly for a Codex launch?"
  - "Go-filled prompt text (mode_rules in loomengine, plan-gate finding text in planparser, darnengine, driftengine, burlerengine, gatefixer, shuttlecli, loomcli, websterengine render): chosen per engine or worded for any engine, with meaning preserved?"
  - "internal/stencilstore engineblock.go and stencilkit: engine block resolution for the launch's engine, behavior when a copy is missing, and consistency with the render callers that now pass an engine."
  - "contracts/stencils/engineterms_test.go scan: closed term list completeness against D2, orch places skipped (stencils/orch, orchrole, orch packages), claudeengine literals excluded, allowlist entries each carry a reason that is true (the literal never reaches a Codex session), and the missing-copy check covers every registered engine."
  - "pattern/PATTERN-stencil-ownership.md: does the diff land the declared change text, and do the edit-directive background edits match D3's statement?"
  - "Test economy: the new tests against the redundancy report; flag any new assertion pinning stencil prose."
---

Seeded from the decision record and the diff stat; no round has been judged yet.
