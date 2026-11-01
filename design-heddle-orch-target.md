# Heddle gains an orch-prompt target, and lym orch rules stores in heddle

`heddle-orch-target` · Task · enhancement, heddle, orch · running · Describe

## Seen (orch, 2026-11-01)

Heddle's targets, in `internal/heddleengine/target.go`, are per engine only: `claude-file`, `codex-file`, `claude-prompt` and `codex-prompt`.
A heddle entry aimed at `claude-prompt` reaches every Claude session in the hub, writers, perches, webster and the tackler alike, so a rule for the orch alone has no place in heddle.
Repository rules for orchs therefore live in a mechanism of their own: `lym orch rules` replaces the hub-wide file `_lym/orch/rules.md` in the Bolt checkout, and each orch also has a local `_lym/orch/rules.md`, which is empty and unused.
Both are inlined into the rendered `role.md`.

## Idea (operator, 2026-11-01)

- Heddle gains an `orch-prompt` target, one new row in the target table, whose passages are appended only to an orch session's system prompt.
- `lym orch rules` stays the operator's verb for the orch rules, and stores them in heddle under that target.
- The hub-wide rules file on Bolt and the per-orch local rules file go away, together with their sections in the rendered role.
