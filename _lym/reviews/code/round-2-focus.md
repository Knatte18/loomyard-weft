---
round: 2
focus:
  - "internal/darnengine/spec_test.go, internal/twilldirective/twilldirective_test.go and internal/websterengine tests: does a row exist per producer that resolves the launch to codex (or a fake engine with seeded block copies) and fails if the producer passed the wrong engine? The webster rows must give Master an engine that differs from the recovery role's and cover the begin-batch fork prompt and the verify-fix prompt."
  - "contracts/stencils/engine/codex/fork-wait.md: does the copy tell the parent to count one final answer per fork, given that wait_agent names no agent and returns once any child's message is pending?"
  - "contracts/stencils/webster/webster-template-master.md: does any Claude-only tool name, such as SendMessage, remain outside a Claude engine block in a stencil rendered for Master's engine?"
  - "Headers of loom-template-discussion, loom-template-plan and raddle-template-agent, internal/loomengine/template.yaml and the DarnInputs.SubagentModel doc in internal/darnengine/spec.go: do any still describe subagent_model as a top-level stencil marker or as the Agent tool's model, where only the Claude subagent-lookups block now declares and uses it?"
  - "contracts/stencils/engineterms_test.go missingEngineBlockCopies: does a stencil whose IncludeNames call fails get reported rather than skipped?"
---
