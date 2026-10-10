---
format: 5
approved: true
language: go
---

# Plan: loop-small-bugs

Fix five small faults in the loop machinery, each in the module that owns it: the review bounce budget that blocks an approved round (#503), the PR-Gate budget prose that states a rule the engine does not have (#526), batten's idle notice counting machine sleep as idle time, shuttle's hold notice firing on the skill-load turn end (#485), and shuttle's wait loop reading `held` beside a session state of `busy` for a whole turn.
The cards follow the decision record's D1 to D5 in order; D4 is split into the prompt-offset read and the hold guard, since each builds and tests on its own.

## Card Index

1 — review-budget-on-bouncer-row — the Burler's two hand-backs for judgment are budget-exempt, so the Bouncer's own check is the segment's one budget on judged rounds
2 — pr-gate-budget-prose — the loom and darn recipe headers and the overview state the PR-Gate episode rule instead of a per-task budget
3 — batten-awake-idle-clock — batten's quiet and api-error idle clocks measure awake time through an injectable awake clock
4 — prompt-offset-before-send — `PromptOffset` is the events-file size read immediately before the prompt is sent
5 — hold-after-prompt-turn-start — a turn end before the prompt's own turn start is logged, never held and never notified
6 — held-wait-ends-at-turn-start — the held wait ends at the next turn start, and the disagreement line says the loop stands

## Shared Decisions

### Decision: turn-starts-from-the-loop-read

- **Decision:** cards 5 and 6 read turn-start signals from the bytes the wait loop's own tick parses, through the engine's `SessionSignalParser`, positioned by offset; neither reads the shadow's fold in `waitshadow.go`.
- **Rationale:** the shadow is display only and a shadow read failure must change no hold or classification; one read serves both the guard and the hold's end.
- **Applies to:** cards 5, 6

### Decision: tests-extend-not-add

- **Decision:** every new case is a row, subtest or assertion in the test that already covers its behaviour, except the helper table test card 3 adds for the gap arithmetic; no package passes its row in `cmd/lyx/testdata/test-budget.yaml`, so the file is untouched.
- **Rationale:** PATTERN-test-economy, and the three packages at risk sit at or near their rows.
- **Applies to:** all cards

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
