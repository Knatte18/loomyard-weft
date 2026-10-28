---
round: 1
focus:
  - "CLAUDE.md, every removed kind-1 rule (D1): grep the shipped stencils, plugin skills and Go code for each one, and settle whether it is still stated or enforced somewhere that reaches the agents that need it, in particular the run-lock rule (no decision, card or step of a run changes the running task's own entry) in the discussion, plan, darn and review stencils, the `deploy` row in `internal/battenshed` and the orch stencil, and the PATTERN overview inlining in the pattern directive stencils."
  - "CLAUDE.md, the kept rules (D2): settle whether every kind-2 rule D2 lists is still present and whether a kind-1 rule survives, in the new notes section, the Docs section and the Terminology bullets."
  - "CLAUDE.md, the new section 'This file holds LoomYard development rules only': settle whether it states the kind split the board entry asks for, and whether it drops the orch-rules routing (hub-wide file versus thread file) without that routing living in the orch stencil."
  - "contracts/stencils/orch/orch-template-role.md, the added memory-store sentence in 'Scratch and notes': settle whether it fits `PATTERN-stencil-ownership`, contradicts a neighbouring rule on `.scratch/`, `.lym/` or git notes, and whether stencil tests in `contracts/stencils` assert on this section."
  - "plugins/ly/skills/board/SKILL.md, the two added lines in 'Writing a body': settle whether they match the rule that left CLAUDE.md, whether the claim that `lym board prune` removes a done entry is borne out by the board code, and whether they conflict with surrounding text."
  - "Scope and commit message: settle whether the diff changes anything D3 and D4 say stays unchanged, and whether the commit message's claim that every other removed rule was already stated or enforced is borne out."
---

Seed focus for a docs-and-stencil diff of three files; no Go code changes, so test-economy and mechanical-gate dimensions are expected to be quiet.
