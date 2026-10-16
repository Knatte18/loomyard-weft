---
format: 5
approved: false
language: go
---

# Plan: Webster fork and pattern-directive fixes

Fix the five Webster fork and pattern-directive findings of the 2026-10-15 stencil intent audit, per the decision record: a symbol `Rename` runs the hub's structure-aware rename tool from `language.yaml`'s new `rename_tools` (cards 1–3, 11, 13, 14, 17); a fork's PATTERN departure travels as a `Pattern-Departure:` commit trailer that Go carries into the digest and `summary.md` (cards 2, 15, 16, 17); the fork spawn and completion wording matches how Webster runs (cards 13, 17); the inline kick-start pack layout and `kickstart_pack_file` go (card 12); and a task declares its PATTERN changes in an optional `## PATTERN changes` section that the gates check and every PATTERN directive renders (cards 4–11, 17).
This task changes no PATTERN entry (D5.5).

## Card Index

1 — language-rename-tools — language.yaml gains the Guard-classified rename_tools list
2 — plan-spec-rename-departure — plan spec names the rename tool, splits the Rename mechanic and owns the departure trailer
3 — planparser-symbol-renames — planparser classifies a plan's symbol Rename pairs
4 — planparser-pattern-changes — planparser parses the overview's PATTERN changes section
5 — discussion-pattern-changes — discussionparser checks and reads the record's PATTERN changes section
6 — discussion-append-pattern-change — AppendDecision appends a PATTERN change bullet and the drop check covers it
7 — decision-add-pattern-form — lyx loom decision add gains the PATTERN-change form
8 — pattern-directive-changes — every PATTERN directive gains the optional declared-changes marker
9 — webster-declared-changes — the Master and recovery directives render the declared changes
10 — review-loom-declared-changes — the Burler, Bouncer and plan writers render the declared changes
11 — plan-gate-context-checks — the plan gates add rename-tool-missing and pattern-changes-mismatch
12 — webster-inline-pack-removal — remove the inline pack layout and retire kickstart_pack_file
13 — implementer-rename-verify-markers — the implementer body renders rename_tools and plan_verify
14 — rename-tool-refusal — begin-batch and recover-batch refuse a symbol Rename with no rename tool
15 — digest-departures — record-batch and recover-batch carry Pattern-Departure trailers in the digest
16 — verify-gate-fix-commits — the verify gate writes the fixer's commits and departures into summary.md
17 — stencil-wording — edit directive, Webster-Review rubric, pattern directive and webster stencil wording

## Shared Decisions

### Decision: declared-changes-as-bullets

- **Decision:** `discussionparser.DeclaredPatternChanges` returns the record's declarations as rendered bullet lines, and every `pattern.Directive` caller passes those lines in.
- **Rationale:** `internal/pattern` stays a leaf (`PATTERN-leaf-packages`) and `internal/discussionparser` stays the record's sole parser (`PATTERN-sole-parsers`), with no conversion type duplicated across callers.
- **Applies to:** cards 5, 8, 9, 10

### Decision: one-commit-log-read

- **Decision:** one read-only `Git.CommitLog` method serves the digest's departures and the verify gate's fix commits.
- **Rationale:** both read the same first-parent non-merge range shape; the read goes through `internal/gitrepo`'s `Repo.Log` from `gitwrap.go`, the one webster file that reads git.
- **Applies to:** cards 15, 16

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
lyx loom lint-comments
