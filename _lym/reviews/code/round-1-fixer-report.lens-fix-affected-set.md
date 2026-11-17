```yaml
lens: fix-affected-set
outcome: reported
```

Searched for set members the fix missed (F1, F3, F4, F5/F7). One twin was missed.

**Findings**

- **LOW** — `internal/loomengine/config.go:494`: the `SubagentModel` field doc still says "the model name the writers pass to the Agent tool for the fresh subagents they hand lookups to".
  - It is the same wording F7 fixed at `internal/darnengine/spec.go:65` and F5 fixed in `template.yaml:9`; a Codex writer ignores the key.
  - Evidence: the `grep "Agent tool"` over `internal` and `contracts` returned this line.

**Searches that found nothing missed**

- F3: `grep SendMessage` over `contracts/stencils/**/*.md`, excluding `engine/claude` and `orch`, returns nothing.
- F4: every stencil naming `subagent_model` now describes the marker as declared by the Claude engine block.
- F1: the stencils that include `engine-fork-spawn`, `engine-fork-wait`, `engine-darn-subagents` or `engine-subagent-lookups` are the burler, darn, loom, raddle, twill and webster files.
  - `burler-cluster-fan` and `burler-fix-self-check` carry fork wording but got no new row. The review's F1 did not name them; recorded as unchecked, not as a finding.
  - `loom-template-discussion`, `loom-template-plan` and `raddle-template-agent` include `engine-subagent-lookups`; F1 named only the twill directive for loom, and no row selects Codex for them either.

**Unchecked:** F2, F6, and F5's `template.yaml` wording are single-site findings, so there was no set to rerun.

**Calls**
1. `grep SendMessage` over the stencils: F3's set.
2. `grep subagent_model` over md, go and yaml: F4's, F5's and F7's set.
3. `grep "Agent tool"` over `internal` and `contracts`: the set for F5's and F7's wording.
4. `grep` for stencils including the fork and subagent blocks, plus Codex rows in the `loomengine` tests: F1's producer set.
