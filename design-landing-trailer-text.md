# landing.yaml trailer_text: an operator-set trailer block for the landing commit

`landing-trailer-text` · Task · enhancement, config, loom

Depends on: [`code-review-fan`](design-code-review-fan.md)

## Seen

Finalize appends exactly one `Co-Authored-By: <value>` trailer to the landing commit, with the value from `landing.yaml`'s `co_authored_by` (`internal/landingshed/config.go`, `internal/summaryparser/summary.go`).
The key refuses an empty value, and its name and the fixed `Co-Authored-By: ` prefix leave the operator no say over what the trailer block holds.

## Operator decision 2026-10-17

- Replace `co_authored_by` with a generic `trailer_text` key in `landing.yaml`.
- Go copies its value verbatim, as the trailer block after the change description, with no prefix added: whoever wants `Co-Authored-By: Claude <noreply@anthropic.com>` writes that whole line.
- The value may hold several lines, for example a `Co-Authored-By` line, a `Created using Loomyard` line and later a Codex line.
- An empty value lands the commit with no trailer block.
- The template default keeps today's behaviour: `Co-Authored-By: Claude <noreply@anthropic.com>`.
- `lyx config reconcile` migrates an existing `co_authored_by` value to `trailer_text` as `Co-Authored-By: <value>`.
- The change description still may not carry trailers of its own; the Describe validation keeps refusing them.
- Update `internal/landingshed/doc.go` and the specs or stencils that name `co_authored_by`.
