---
round: 1
focus:
  - "internal/heddleengine/target.go `promptTargetsFor` and render.go `Passages`: does `orch-prompt` reach an orch session alone, and does `agentname.MatchesRole` match the strand role values `spec.Role` carries for orch sessions (including numbered siblings) and no other role? Does every other Passages caller of the changed signature still pass a role?"
  - "internal/shuttleengine/claudeengine/claudeengine.go `Prepare` and the other `StandingPassages` callers (codex engine, any other launch path): is the role passed from a field that is set for orch launches, and does an orch session on codex or any other engine still get the right passages, with an entry aimed at both `claude-prompt` and `orch-prompt` delivered once?"
  - "internal/orchcli/rules.go `applyOrchRules` and `heddlecli.ChangeStore`: does the verb meet the Idea (store under `orch-prompt`, empty draft clears, blank-but-whitespace draft, missing entry on clear)? Does `ChangeStore` go through the same lock, validation and sync as `lym heddle set`/`remove`, and are its warning and error paths surfaced in the envelope?"
  - "Removal of the hub-wide and per-orch rules files: is every reader, path (`orchengine.Paths`, `orchroute/paths.go`), `RenderRoleFile` argument, stencil marker and section in orch-template-role.md gone with no dangling reference, and does the role text now say the rules arrive in the system prompt? Is existing-hub behavior for an already-written Bolt rules file stated or handled as the Idea implies?"
  - "Scope against the Goal: changes in cmd/lym/main.go, configcli test, PATTERN.md, pattern/PATTERN-heddle-instructions.md, orchengine watcher/state/resumecontext and the doc.go files: does each follow from the Idea, and do the docs the repository's rules name for this kind of change land in the diff?"
  - "Test economy in render_test.go, rules_test.go, rules_integration_test.go, prompt_test.go, prepare_test.go, start_test.go: new tests duplicating existing coverage, and the new behaviors (role-narrowed targets, ChangeStore, claude Prepare passing the role) each covered."
  - "Commit message claims against the diff, notably 'heddlecli.ChangeStore: Bolt write lock, then the board's detached Bolt sync' and 'An empty draft removes the entry'."
---

Seed focus for round 1. The decision record holds the Goal alone, so the amended board entry is the measuring stick; no `_lym/landing/summary.md` was checked.
