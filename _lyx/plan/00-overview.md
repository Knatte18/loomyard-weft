---
format: 5
approved: false
language: go
---

# Plan: burler-cost — split a Burler round into a reviewer and a fixer

A Burler round today is one agent that reviews and then fixes.
This plan splits it into two shuttle runs started together inside `burlerengine`: a cheap reviewer that writes only the review, and a stronger fixer that orients while the review runs, waits on a Go-written marker through `lyx burler await-review`, then validates and fixes the findings, disputing one only with evidence that its premise is false.
`loom.yaml`'s `review` and new `fix` keys pick each half's model per round.
The plan lands the independent pieces first (the shuttle probe, the marker path, the model lists, the wait verb, the stencils), then the engine split with all its callers, then the two-half resume.

## Card Index

1 — shuttle-probe — `Runner.ProbeGated`: AttachGated's probe returning the handle unwaited
2 — ready-marker-path — `burlermarker.Path`: the one review-path-to-marker derivation
3 — per-round-models — `review`/`fix` scalar-or-list keys, resolved per round, recipe `model`/`effort` keys removed
4 — await-review-verb — `lyx burler await-review <marker>`: read-only, capped poll
5 — round-stencils — reviewer and fixer orchestrator stencils; the fixer's `Disputed` section
6 — two-agent-round — `Engine.Run` starts both halves and joins them through the marker; callers follow
7 — two-half-resume — `ProbeRound`/`Resume` and the producer's two-strand probe and retry

## Shared Decisions

- The marker path is derived only by `burlermarker.Path` and told to the engine on `Profile.ReadyMarkerPath`; `burlerengine` derives no path (`PATTERN-told-geometry`).
- Stopping a half is always `StrandRemover.RemoveStrandIfLive` on its strand guid; a failed stop surfaces as `ErrHalfNotStopped` and is never archived over or retried.
- `join` is the one place the round's failure rules live; a fresh `Run` and a resumed `Resume` both end through it.
- New behaviour is tested at `burlerengine`'s and `shedadapters`' public surfaces with fakes (`PATTERN-test-economy`, `PATTERN-test-speed`); the `llm`-tagged smoke tests only move to the new signatures.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
