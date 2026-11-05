# Landing credit block names Codex when a Codex session ran

`codex-credit-line` · Task · enhancement, landing, codex · running · Darn-Write

## Seen (2026-11-05)

The Codex trial run `codex-greet` in the sandbox hub `lyt` landed with a Codex writer, yet its landing commit's credit block named LoomYard and Claude only.
`credit_engines` in `internal/landingshed/template.yaml` lists `claude` alone, so an engine that ran a session but has no entry adds nothing.

## Wanted

- `credit_engines` in the landing template gains `codex: "Codex https://github.com/openai/codex"`, so a task with a Codex session credits Codex in the same plain-line credit block as Claude.
- `TestLoadConfig` (or the test in `internal/landingshed/config_test.go` that pins the template's `credit_engines`) expects both entries.
- No `Co-Authored-By` trailer or any other trailer, for any engine: the credit block stays plain lines (operator, 2026-11-05).
