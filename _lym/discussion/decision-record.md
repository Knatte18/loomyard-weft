## Goal

```markdown
## Seen (orch, 2026-11-01)

The five heaviest Webster forks read one file in three to five slices over as many turns, such as `sed -n 200,380p`, `380,560p` and `560,760p` of `internal/loomcli/decision_test.go` in three turns, and re-read the same files in four to seven separate turns.
About 80 of their 208 single-call read turns could have been issued a turn earlier, since the path or line range was already known.

## Idea

The Webster implementer stencil, and the batching sentence every agent gets, ask for a file to be read whole, or for every span the agent needs from it, in one message, and for a file read once to be kept in mind rather than read again.
Note `edit-with-edit-tool` removes part of this, since the Read tool reads whole files or large spans.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-01): Read-whole wording moves into the shipped heddle entry

Decision: The merge resolver carries this branch's read-whole wording into the lym-batching entry of internal/heddleengine/shipped.json and updates the pinned sentence in the ShippedBatchingEntryIsPinned subtest of internal/heddleengine/store_test.go.

Rationale: Main moved the batching sentence from claudeengine into heddle's shipped layer; dropping the wording would leave this task's change shipped nowhere.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
