---
format: 5
approved: false
language: go
---

# Plan: forkaudit, a repeatable Webster fork read audit

Add `tools/forkaudit/`, a bash and jq tool that finds Webster implementer fork transcripts by date window or run, classifies each fork's pack layout from its prompt, cross-checks it against the begin-batch trace line, and writes a per-arm markdown report, with optional pack-correctness and plan glyph-use sections.
It replaces the hand-run pipelines of the two kept glyph audits, which stay unchanged.
Go integration tests in the same directory run the script on synthetic fixtures, and `PATTERN.md` admits the script's run-record literals.

## Card Index

1 — cli-and-output — script command line, output and work-directory bounds, project discovery, header and zero-fork report; PATTERN line; test package
2 — fork-records — candidate discovery, per-fork extraction, arm marker table, samples and per-fork tables
3 — arm-tables — pack-use, arm comparison and quartile tables with the missing-metric rule
4 — trace-cross-check — batch resolution from the records repo and the trace-line cross-check
5 — pack-check — optional pack correctness at batch start behind --pack-check
6 — plan-glyph-use — optional plan glyph use behind --plan

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
git diff --quiet "$(git merge-base HEAD main)" HEAD -- docs/research/glyph-audit-2026-10-13 docs/research/glyph-audit-2026-10-15
