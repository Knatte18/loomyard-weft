---
format: 5
approved: true
language: go
---

# Plan: Plan-Write respawn carries the prior plan forward; the Claude engine announces its tool denies

A respawned Plan-Write session starts from an empty plan directory because the fresh-spawn rotation archives the prior plan and nothing tells the new session where it went.
Separately, the Claude engine's `PreToolUse` hooks refuse the `Agent` tool (and `AskUserQuestion` in autonomous runs) without the session ever being told, so it loses work to refused calls.
This plan makes `claudeengine` announce every standing deny through `--append-system-prompt` on the launch and resume lines, widens `shedadapters.SingleLLMProducer`'s fresh-spawn preparation so it can append to the prompt, and has the Plan-Write rotator append a stencil-rendered prior-plan block naming the archive directory and the moved files.

## Card Index

1 — claudeengine-deny-notice — build the one-line notice announcing each deny `buildSettings` installs
2 — claudeengine-append-system-prompt — pass the notice via `--append-system-prompt` on launch and resume, with docs
3 — singlellm-prompt-amendment — let the fresh-spawn preparation append to the respawned session's prompt
4 — prior-plan-stencil — add the `loom-template-prior-plan` stencil and point the plan template at it
5 — loomengine-prior-plan-block — render the prior-plan block from the stencil
6 — rotator-prior-plan-amendment — have the Plan-Write rotator return the block naming the archive it created

## Shared Decisions

- The notice constants live beside the steer constants in `settings.go` and share their forbidden-character check, extended to line breaks; the assembled notice is one line.
- The preparation seam's type is `func() (string, error)`: returning text rather than receiving the spec makes the amendment append-only by construction.
- The prior-plan block is rendered in `loomengine`, which already reads the loom stencils, so `loomshed`'s import allowlist does not change.
- The stencils directory reaches the rotator told, from `shedrecipe.Env.StencilsDir`.

## verify:

```
go build ./...
go test -tags integration ./...
go vet -tags smoke ./...
```
