# Round 2 fixer report — discussion

All four findings were checked against the code and the host template, and all four held.
Every one is fixed.
`lyx loom validate-discussion` passes after the edits.

## Fixed

- **F1 (MEDIUM, design): two write scopes in the twill prompt.**
  The template's `## What you may write` opens with "The only files you create or modify in this session are the two named in Step 5", and says "outside those two files" further down.
  D2's second marker now sits directly below the file list.
  For `twill` it renders a write-scope paragraph saying the ready marker is a third file the writer creates, along with its parent directory, and that every "two files" or "those two files" in the section includes the marker.
  The section's own sentences are unchanged, so `single` stays byte-identical.
  D3 adds the sibling host-neutral write-scope stencil that holds the paragraph, rendered by the same package.
  The acceptance criterion no longer says "no other path", which was false because the section names forbidden paths; it now pins the widened scope.
- **F2 (MEDIUM, design): D6's bound depends on config.**
  This is confirmed in `internal/shuttleengine/claudeengine/settings.go`: the fork-only hook installs only when `cfg.ClaudeDenyAgentTool` is true, and `internal/shuttleengine/template.yaml` defaults it to `true`.
  D6's bound now says the fork-only refusal holds while `claude_deny_agent_tool` is true.
  With it false, no Agent hook is installed, as for `single`, but the webster fork guard stays.
- **F3 (MEDIUM, design): the relaunched-writer rule assumed a death after the marker.**
  D4 now says a relaunched writer resumes at the first of steps 3 to 8 it had not finished and never spawns again.
  Step 7 records each fork that never returned as "no report", or records that no forks were spawned at all.
  D4 also names the writer's next turn: a gate re-prompt or a message from its parent, or else the run's existing timeout.
  The matching acceptance criterion and the ledger's Q17 are updated to match.
- **F4 (LOW, consistency): the support log contradicted D9 on PATTERN-daemon-wakeups.**
  The rejected 200 ms alternative now gives D9's basis.
  One stat a second is the rule's floor for the waits it governs, and the rule does not bind a call that has its own deadline.

## Disputed

None.

## Deferred

None.
