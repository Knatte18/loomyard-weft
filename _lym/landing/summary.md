# Credit Codex in the landing credit block when a Codex session ran

A task whose sessions ran under Codex landed with a credit block naming only LoomYard and Claude, because the landing template's `credit_engines` listed `claude` alone and an engine with no entry adds no line.

## What changes

- `internal/landingshed/template.yaml`: `credit_engines` gains `codex: "Codex https://github.com/openai/codex"`.
  A task with a Codex session now gets a ` - Codex https://github.com/openai/codex` line in the same plain-line credit block as Claude, on both the landing commit and the pull request's body.
- `internal/landingshed/config_test.go`: `TestLoadConfig` expects both the `claude` and `codex` entries from the template.

## Design decisions

- The credit stays a plain line; no `Co-Authored-By` or other trailer is added for any engine.
- Codex has no noreply address to cite, so its item names the project's URL, in the same form as the LoomYard item.
- The key's meaning is unchanged, so the change ships no `hubreconcile` reinterpretation.
