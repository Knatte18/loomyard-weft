# Heddle gains an orch-prompt target, and lym orch rules stores in heddle

A repository's orch rules now live in heddle and reach an orch session alone, appended to its system prompt.
Before this, heddle's prompt targets were per engine only, so a `claude-prompt` entry reached every Claude session in the hub.
The orch rules therefore had a mechanism of their own: a hub-wide `_lym/orch/rules.md` on Bolt, an unused per-orch `_lym/orch/rules.md`, and two role-file sections that inlined them.

## Heddle: the orch-prompt target

- A prompt target row in `internal/heddleengine/target.go` now names the sessions it reaches by `Engine`, by `Role`, or both.
  `claude-prompt` and `codex-prompt` keep their engine.
  The new `orch-prompt` row (`heddleengine.OrchPromptTarget`) names `agentname.RoleOrch`, matched with `agentname.MatchesRole`, so a numbered orch such as `orch-2` is reached too.
- `Renderer.Passages(boltDir, engine, role)` returns, in render order, every entry aimed at any prompt target that reaches the session.
  An entry aimed at both `claude-prompt` and `orch-prompt` appears once.
- Shuttle's `PassageReader` and `StandingPassages` take the session's role.
  The claude engine passes `spec.Role`, which the orch's launch spec sets to `orch`.
- `lym heddle show orch-prompt` prints the orch passages, and `lym heddle set` accepts the new target.

## lym orch rules

- `lym orch rules --from <file>` writes the draft as the heddle entry `orch-rules`, aimed at `orch-prompt` alone.
  An empty draft removes the entry.
  A new entry takes render order 1000, so it comes after the passages every session shares.
  An existing entry keeps its order.
- The write goes through heddle's own write path: the new `heddlecli.ChangeStore` makes the same write as `lym heddle set`, under the Bolt write lock and followed by the board's detached Bolt sync.
  So the verb no longer pulls, commits and pushes in the foreground.
  Its envelope now carries `slug` and `cleared`, plus `warning` when the sync did not start.
  The old `path`, `committed` and `unpushed` fields are gone.
- `PATTERN-heddle-instructions` and its `PATTERN.md` line now let another module's verb write the store, but only through that shared write path.

## The rules files go away

- `orchengine.Paths` loses `HubRulesPath` and `OwnRulesPath`, and `orchroute` loses `RulesRel`.
- `RenderRoleFile` takes only the stencils directory, the path and the command index.
- The role stencil drops the `hub_rules` and `orch_rules` markers along with the `Rules files`, `Hub-wide rules` and `This orch's rules` sections.
  A short `Orch rules` section takes their place: the rules arrive appended to the system prompt, add to the role, and a contradiction is raised to the operator.
  The stencil's other mentions of the rules file now point at `lym orch rules`, and a draft starts from `lym heddle get orch-rules`.

## Design decisions

- **Matching by strand role, not engine.**
  The orch may run on any engine, so `orch-prompt` names no engine.
  Only the claude engine appends standing passages today, as it already did for `claude-prompt`.
- **Existing rules files are left on disk.**
  lym no longer reads either file but deletes neither, so the hub's current orch rules are not lost before the orch moves them into heddle.
  Per the `orch-instructions` project, the orch moves them with `lym orch rules`, and that move is no task.
- **Tests.**
  A tier0 table covers the orch-rules store change: a new entry, a replacement that keeps its order, a blank draft clearing it, and a blank draft with no entry.
  A tier1 test drives `lym orch rules` over a real hub into `heddle.json` and back out.
  A heddleengine table covers which sessions `orch-prompt` reaches.
