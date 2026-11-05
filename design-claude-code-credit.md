# Credit Claude Code, not Claude, in the landing credit block

`claude-code-credit` · Task · enhancement, landing, claude · running · Code-Bouncer

## Wanted (operator, 2026-11-05)

The landing credit block lists `Claude <noreply@anthropic.com>` when a Claude session ran; the engine is Claude Code, so the item reads `Claude Code <noreply@anthropic.com>`.

- Change the `claude` item of `credit_engines` in `internal/landingshed/template.yaml` and the tests in `internal/landingshed/config_test.go` that pin it.
- Check that no hub layer or materialized config keeps the old text, and that `hubreconcile` needs no reinterpretation, since the key's meaning is unchanged.
- No trailer of any kind is added.
