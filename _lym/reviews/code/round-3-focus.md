---
round: 3
focus:
  - "internal/loomengine/plan.go (PlanTable passes the lead's choice.Engine to the twill directive) and internal/loomengine/discussiontable.go (DiscussionTable passes chairChoice.Engine to the lens directives): does a test put the lead on codex with lenses set and fail if the directive took the default or an advisor's engine, by asserting the lead prompt carries Codex fork wording and not Claude's?"
  - "internal/webstercli/beginbatch.go beginDeps resolves the Master role's engine through c.engines.EngineName: does a test give Master and Recovery different engines and fail if begin-batch read Recovery's or the default engine, by asserting the written fork prompt carries Master's engine wording?"
  - "contracts/stencils/engineterms_test.go claudeLiteralAllowlist and goLiteralTermViolations: does each allowlist entry admit only the one literal whose reason it states, keyed by the enclosing declaration, rather than every literal in the file naming the term?"
  - "contracts/stencils/webster/webster-template-master.md: does any Claude Code mechanism, such as a backgrounded Bash command with a completion notification, or any header text naming the Agent tool, remain outside a Claude engine block in a stencil rendered for Master's engine?"
  - "cmd/lym/agentmessaging_test.go and _lym/landing/summary.md: do the allowlist doc and fixture names describe the Claude fork-message engine block that the one admitted row keys, and does the summary list every new engine block the diff registers in contracts/stencils/stencils.go?"
---

Focus restates round 2's open findings as questions to settle; the review judges the diff on its own.
