---
format: 5
approved: true
language: go
---

# Plan: Twill for Plan-Write at opus[medium]

Build the twill producer for Plan-Write: under `loom.yaml`'s new `plan_producer: twill`, the plan writer forks itself once per lens of a new four-lens `twill-plan` fan before it writes, writes and self-checks the plan, marks it ready, takes the forks' findings in and records them in a tracked `lens-review.md`, rejecting a BLOCKING or MEDIUM finding only with a cited code read.
Beside it, a deterministic `fabric-vocabulary-name` plan check refuses a declared name the fabric-vocabulary scan would refuse, backed by the scan's predicates moved into a production leaf package.
D11, selecting twill at `opus[medium]` for this repository, is no card: per the parent's recorded decision ("D11 lands after deploy, outside the cards"), the orch runs `lyx config loom --set plan_producer=twill --set 'plan=opus[medium]'` after this task lands and deploys.

## Card Index

1 — fabricvocab-package — move the fabric-vocabulary predicates into the stdlib-only `internal/fabricvocab` (D10)
2 — vocabulary-name-check — add `planparser.CheckFabricVocabularyName` (D9)
3 — index-runs-vocabulary-check — append the check in the plan index's format, rework and dispatch validation, with spec and help text (D9)
4 — twill-host-values — the twill directive takes spawn point, rejection and severity rules and a reviewed-files phrase from its host (D4)
5 — twill-plan-fan — add the `twill-plan` fan and its four lenses to the burler template (D8)
6 — plan-producer-key — add `plan_producer`, `single` by default (D1)
7 — plan-ready-marker — the plan ready marker, removed before every fresh Plan-Write spawn (D6)
8 — lens-review-file — admit `lens-review.md` in the plan directory (D7)
9 — plan-twill-directive — Plan-Write renders the twill directive and authorizes forks under `twill` (D2, D3, D5, D8)

## Shared Decisions

- No production file outside the fabric-vocabulary owner set spells a fabric side token, and no test file outside it either: tests build a side-token name from `fabricvocab.SideTokens()`, and host-geometry rows spell `hostBranch`, which the host rule leaves to non-test files.
- The plan host's rejection rule and spawn point carry D5's and D3's content in the prompt's second-person voice, and its severity rule and per-fork line are rendered verbatim as the record states them (card 9), since the bench file they come from is unreadable to the writer.
- `single` keeps today's prompt bytes: every new template marker is optional and sits on a line that would otherwise be blank.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
