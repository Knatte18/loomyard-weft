## Goal

```markdown
## Seen (board tidy and orch, 2026-11-07)

Heddle's `claude-settings` target (`72390eb3`) renders `.claude/settings.local.json` in every hub worktree, triggered by fabric, so a session the operator starts by hand gets lym's settings.
Lym's own sessions also get a per-session `--settings` document that `internal/shuttleengine/claudeengine` builds at each launch (`settings.go`, `scope.go`).
Some of what that document sets is the same for every session of a worktree, such as `attribution` and `promptSuggestionEnabled: false`, so those settings are spread over two files.
That commit also left model and effort defaults to be added to the target later.

## Wanted (operator, 2026-11-07)

The point: what lym sets in Claude's settings is steered from one place per kind and can be read where it belongs.

- Every setting that is the same for every session of a worktree is rendered by heddle into `.claude/settings.local.json` and no longer set in the per-session document.
- Every setting that differs per session or role stays in the per-session `--settings` document: hooks that name one session's own paths, such as the events file, and anything bound to one role.
- Per-role differences in `enabledPlugins` and `permissions.deny` are checked against the operator's rule that every agent keeps the same tool set; a difference that rule removes goes, and a setting that is then the same for every session moves to heddle.
- The machine-wide model and effort defaults are rendered into the heddle file from the `agents.yaml` that `agent-skills-and-effort` builds, so each value has one source; per-role effort stays on the launch.
- Planned to follow the landing of `agent-skills-and-effort`, which changes the same launch code.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-07): Heddle renders HEDDLE.md in Bolt

Decision: Scope is widened (operator, 2026-11-07): every heddle write also renders HEDDLE.md in the Bolt checkout, beside heddle.json, as board writes render README.md. It is a pure render of what heddle holds: every entry, the shipped ones and the hub's own, each with its name, its targets, where it comes from (shipped or hub) and its text, grouped by target, in clean readable markdown. A JSON-object entry for the claude-settings target shows its object as a fenced json block. It adds no state of its own and is rewritten whole at each render.

Rationale: The operator wants to read in one place everything heddle sets, the Claude settings included, without reading heddle.json or the binary's shipped entries.

### Added after Discussion (parent, 2026-11-07): Only the effort default is rendered

Decision: Review finding F1 of round 1: the heddle file renders the effort default from agents.yaml and no model default. agents.yaml holds no model key, and this run adds none; the board entry's 'model and effort defaults' rested on a false premise. No follow-up is filed for a model key.

Rationale: Lym sets each role's model through its own model spec at launch, and a session the operator starts by hand keeps the model the operator picks, so a hub-wide model default has no caller that needs it.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - model-default-not-rendered (class decision, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
