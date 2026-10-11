---
format: 5
approved: true
language: go
---

# Plan: twill, the self-forking Discussion writer

Add `twill` as a third `discussion_producer` beside `single` and `seats`: the single writer's spec with fork subagents authorized and a twill block rendered into the discussion template.
The block comes from a reusable, host-neutral directive stencil rendered by a new small package; the lenses come from a new `twill-discussion` fan in the burler lens library; the forks wait on a ready marker through `lyx shuttle await`, the general keep-cache-warm wait any agent can use, which `lyx burler await-review` now shares its poll with.
The marker lives under `.lyx` and Go removes it before every fresh writer spawn.
The shed engine, the seat engine and the `single` and `seats` behaviour are untouched.

## Card Index

1 — shuttle-await-poll — one shared capped file poll in `shuttleengine`, `await-review` rewired onto it
2 — shuttle-await-verb — `lyx shuttle await <path>... [--cap]`, the general wait in the shuttle subtree
3 — twill-lenses — `premises`, `board-fit` and the `twill-discussion` fan in the embedded lens library
4 — twill-stencils — the host-neutral twill directive and write-scope stencils, registered and pinned
5 — twilldirective-package — `internal/twilldirective` renders both stencils from a told directory
6 — discussion-template-twill-markers — two optional markers in the discussion template, threaded through the composer and the chair table
7 — twill-config-value — `discussion_producer: twill` loads; the unknown-value error names all three
8 — discussion-ready-marker — the ready marker's path accessor and its removal, under `.lyx`
9 — discussion-spec-twill — `DiscussionSpec` renders the twill block, resolves the fan and sets fork subagents for `twill`
10 — discussion-marker-fresh-spawn — Go removes a stale ready marker before a fresh Discussion-Write spawn

## Shared Decisions

### Decision: poll-in-shuttleengine

The shared poll is a function of `internal/shuttleengine`, which both CLIs import, over the package's existing `Clock` seam, so its test injects a clock and no CLI test waits out a real interval.

### Decision: twill-is-a-spec-variant

The Discussion-Write row keeps the single writer's engine, name, gates, routing, output files, timeout, role, segment and skills; only the rendered prompt and `ForkSubagents` differ, so `loomrecipe` and the shed engine never learn the value.

### Decision: host-values-live-in-loomengine

Every Discussion-specific literal the directive needs (the step names, the self-check command, the disposition place, the rubric name) is a value `internal/loomengine` builds; the stencils and `internal/twilldirective` name none, so Plan-Write can render them later with its own values.

### Decision: marker-removal-after-the-probe

The marker is removed in the single-LLM producer's fresh-spawn preparation, which runs only after the attach probe finds no live writer, so an attach and a relaunch keep a live writer's marker.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
