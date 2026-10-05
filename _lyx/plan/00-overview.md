---
format: 5
approved: true
language: go
---

# Plan: agent-command-wrapping

Agents stop wrapping their commands because lyx removes both reasons they wrap.
Every lyx-spawned Claude session gets `/dev/null` as its Bash tool's default stdin through a `CLAUDE_ENV_FILE` instead of a per-call PreToolUse hook.
`lyx shed step` prints a short envelope that names its trace and its full-envelope record, and the driver stencil shows that bare call as the one way to step.

## Card Index

1 — claude-env-file — replace the Bash stdin rewrite hook with a CLAUDE_ENV_FILE that Prepare writes and both pane lines export
2 — short-step-envelope — step prints a short envelope, writes the full one to its record first, and gains --full
3 — driver-bare-step — the driver stencil shows `lyx shed step {{.run_id}}` as the whole command and drops the cd-subshell rule

## Shared Decisions

- The decision record's acceptance criteria include a live check that no card can run: a Claude session launched by a `./deploy-dev` binary reports `/proc/$$/fd/0` as `/dev/null` in a Bash call and its `settings.json` carries no stdin hook.
  It is recorded in the PR, not in a card's verify.
- Tests in `internal/shuttleengine`, `internal/shuttlecli` and `internal/shedcli` are owned by the test-suite prune tasks; the cards touch only the tests that cover the changed behavior.

## verify:

```bash
go build ./...
go test ./internal/shuttleengine/... ./internal/shuttlecli/... ./internal/shedverbs/... ./internal/shedcli/... ./internal/loomcli/... ./internal/battencli/... ./cmd/lyx/...
go test -tags integration ./internal/shuttleengine/... ./internal/shuttlecli/... ./internal/shedverbs/... ./internal/shedcli/... ./internal/loomcli/... ./internal/battencli/... ./cmd/lyx/...
go vet -tags smoke ./internal/shuttlecli/... ./internal/loomcli/... ./internal/shedcli/...
```
