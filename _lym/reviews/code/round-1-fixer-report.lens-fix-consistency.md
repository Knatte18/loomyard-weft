```yaml
lens: fix-consistency
outcome: reported
```

**Findings**

- **LOW** — `contracts/stencils/engine/codex/fork-message.md` against the unchanged `webster-template-master.md` lines 95–97, 105, 144, 188 and others.
  - The Codex copy of the new `engine-fork-message` block sends the message "addressed to the task name that `spawn_agent` returned."
  - The sentence that follows it, "on the fork's next completion notification, call `record-batch` again", and the many other "completion notification" lines of the Master stencil still use the Claude Code wake model, outside any engine block.
  - The F3 fix removed `SendMessage` from the stencil but left this wake wording, so a Codex Master still reads Claude-specific text there.
  - The review did not name it, and the closed term scan does not catch it, so it is LOW and may be a deliberate scope limit.

- **LOW** — the new `fork-message` blocks against the other engine blocks.
  - No list of engine blocks outside `stencils.go` mentions them; both copies are registered there and the gate was green.
  - No stale sibling found; a non-finding, kept as a record of the check.

**Checked and consistent**

- The `engine-fork-message` include in `webster-template-master.md` line 188 matches the Claude copy's wording.
- The Codex copy reads grammatically after "that batch's fork".
- The F4 header claim holds for all three stencils: each includes `engine-subagent-lookups`, whose Claude copy declares the marker.
- The F5 `template.yaml` comment and the F7 `DarnInputs.SubagentModel` doc agree with each other and with the Codex copies.
- The F2 Codex `fork-wait` wording matches the review's proposed text.

**Tool calls**
1. A grep for `engine-subagent-lookups` and `engine-darn-subagents` includes in `contracts/stencils`: to check the F4 header claim.
2. A grep for `fork-wait` and `fork-spawn` across the repo: for any other place listing the engine blocks.
3. A grep for `question-tool` and `absolute-paths` outside the stencil families: for any doc listing the blocks.
4. A grep for `SendMessage` and `completion notification` in `webster-template-master.md`: for the other Claude wake wording.
