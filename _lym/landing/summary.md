# Heddle gains an orch-prompt target; lym orch rules stores in heddle

An orch's repository rules now live in heddle, aimed at a new `orch-prompt` target that reaches orch sessions only.
Before, a heddle entry aimed at `claude-prompt` reached every Claude session in the hub, so orch-only rules had their own mechanism: a hub-wide `_lym/orch/rules.md` on Bolt and an unused per-orch copy, both inlined into the rendered `role.md`.

## What changes

- `internal/heddleengine/target.go`: `orch-prompt` is one new row in the target table.
  `Target` gains `Engine` and `Role` fields that narrow which sessions a prompt target reaches; `claude-prompt` and `codex-prompt` now name their engine, and `orch-prompt` names the orch role, matching numbered siblings of it.
- `Renderer.Passages`, `shuttleengine.PassageReader` and `StandingPassages` take the session's strand role beside its engine.
  An entry aimed at several matching targets gives its text once.
- `lym orch rules --from <file>` keeps its name but now writes the heddle entry `orch-rules`, aimed at `orch-prompt` alone.
  A new entry takes render order 1000, an existing one keeps its order, and a blank file removes the entry.
  The envelope carries `slug`, `cleared` and, when Bolt sync did not start, a `warning`.
- The write goes through the new `heddlecli.ChangeStore`, heddle's one write path (Bolt write lock, then the detached Bolt sync); `lym heddle set` and `remove` share it.
- The hub-wide rules file, the per-orch local rules file, their `hub_rules` and `orch_rules` stencil markers, `readRules` and the path helpers are removed.
  `RenderRoleFile` no longer takes rules paths.
- A rules file left at `_lym/orch/rules.md` is no longer read.
  The `lym orch rules` help says so and how to carry the content over: copy it to a draft and run `lym orch rules --from` on it.

## Design decisions

- The rules stay behind `lym orch rules` as the operator's verb; only the storage moves, so the verb's `--from` interface is unchanged but its envelope is not (`path`, `committed`, `unpushed` are replaced).
- Role matching uses `agentname.MatchesRole`, so orch strands such as numbered siblings receive `orch-prompt` passages.

## To check by hand

- Existing hubs with rules in `_lym/orch/rules.md` lose them silently at the prompt level; only the `lym orch rules` help names the carry-over.
  Check that this is acceptable, or whether the first orch launch should warn when such a file exists.
- Run `lym orch rules --from <draft>` in a real hub, then start an orch and a non-orch Claude session: the rules should appear in the orch's system prompt only, and `lym heddle get orch-rules` should print them.
- Check that the Bolt sync commits and pushes the `heddle.json` change after the verb.
