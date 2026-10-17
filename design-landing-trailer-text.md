# landing.yaml trailer_text: an operator-set trailer block for the landing commit

`landing-trailer-text` · Task · enhancement, config, loom, shuttle

Depends on: [`code-review-fan`](design-code-review-fan.md)

## Seen

Finalize appends exactly one `Co-Authored-By: <value>` trailer to the landing commit, with the value from `landing.yaml`'s `co_authored_by` (`internal/landingshed/config.go`, `internal/summaryparser/summary.go`).
The key refuses an empty value, and its name and the fixed `Co-Authored-By: ` prefix leave the operator no say over what the trailer block holds.

## Operator decision 2026-10-17

- Replace `co_authored_by` with a generic `trailer_text` key in `landing.yaml`.
- Go copies its value verbatim, as the trailer block after the change description, with no prefix added: whoever wants `Co-Authored-By: Claude <noreply@anthropic.com>` writes that whole line.
- The value may hold several lines, for example a `Created using Loomyard` line; its lines land on every commit.
- A second key, `engine_trailers`, maps an engine name to its trailer text, for example `claude: "Co-Authored-By: Claude <noreply@anthropic.com>"` and later `codex: "Co-Authored-By: Codex <...>"`.
  At landing Go includes an engine's text only when that engine ran at least one session in the run, read from the run's own records of the sessions it spawned, with the engine taken from the model spec, since model aliases encode the engine.
  A run that used only Codex therefore lands with no Claude line.
- Either key can be turned off: an empty `trailer_text` adds no fixed lines, and an empty `engine_trailers` map, or an engine left out or set to empty text, names no engine even when it ran.
  Both empty lands the commit with no trailer block.
- The template default keeps today's behaviour: `trailer_text` empty and `engine_trailers` holding the `claude` line `Co-Authored-By: Claude <noreply@anthropic.com>`.
- `lyx config reconcile` migrates an existing `co_authored_by` value to `engine_trailers.claude` as `Co-Authored-By: <value>`.
- The change description still may not carry trailers of its own; the Describe validation keeps refusing them.
- Update `internal/landingshed/doc.go` and the specs or stencils that name `co_authored_by`.
