## Goal

```markdown
## What was seen (2026-11-22)

The hub gate.yaml held a key the hardener rebuild (2dbfdc31) retired, `gates.harden-live-copies`, with no `scope`.
`lym s` then failed to resume the orch session: claudeengine refused to prepare the orch launch over that one gate key, so the hub had no orch until the key was removed by hand.
`lym config` has no verb to remove a key, so only the interactive editor could fix it.

## Idea

- An orch or other session that runs no gate should not refuse its launch over a gate key it never uses; warn and launch.
- The gate reinterpretation that retires `harden-live-copies` should also cover an entry with no scope.
- Give `lym config` a non-interactive way to remove a key.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-22): Scope-less harden-live-copies removal runs under a new reinterpretation ID

Decision: Register the widened rewrite under a new ID, a constant beside RetiredCopiesReinterpretationID in internal/gateslot/reinterpret.go, for example gate-harden-live-copies-scopeless-retired, so a hub past the first marker runs it once more; update the summary's design decision to match.

Rationale: Hubs on other machines ran the binary that wrote the first marker and may still hold a scope-less harden-live-copies key; a manual --unset on each hub is the step that already failed the orch launch here, and the rewrite leaves a file with no such entry unchanged.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
