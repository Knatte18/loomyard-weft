---
round: 1
focus:
  - "internal/planglyph/doc.go and contracts/specs/loom-plan-spec.md: both sides edited different lines, so check the merged prose reads coherently. Do the task's 'invalid machine override' wording for a failed slot load and the parent's .scratch/ done-check exemption each stay true beside the other?"
  - "internal/shellguard/rows.go and its integration tests (parent) against the task's machine-local gate config in internal/gateslot, internal/configengine and internal/gatecli: does any parent row now refuse, or fail to cover, a command or path the task's machine layer relies on? Settle whether the task's code still passes the parent's new guard rows."
  - "internal/shuttleengine/claudeengine settings.go and testdata/settings-golden.json (parent) against the task's config-layer changes: does anything the task changed feed the generated settings, so the merged golden file or the hook command tests assume the pre-merge behavior?"
  - "tools/deploy/main.go (parent) against the task's gate-slot and machine-config changes: does deploy still build, locate and install what the task's new machine layer expects, and does the task's CLAUDE.md-level or PATTERN-config-layers invariant hold for the parent's deploy changes?"
  - "pattern/PATTERN-config-layers.md, PATTERN-hub-wide-config.md and PATTERN-durable-vs-ephemeral-state.md (task) against the parent's new code in shellguard, claudeengine and deploy: does any parent addition read or write config state in a way those invariants now forbid?"
---

Seed rationale: the only textual overlaps are one-line edits in `planglyph/doc.go` and the plan spec, which merged cleanly.
The semantic risk sits in the parent's shellguard, claudeengine settings and deploy changes meeting the task's machine-local config layer.
