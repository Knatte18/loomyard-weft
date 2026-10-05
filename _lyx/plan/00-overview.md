---
format: 5
approved: true
language: go
---

# Plan: Self-reported issues say something

lyx's Go code stops filing GitHub issues: the empty Tier 1 issues (#366–#382) came from a filer that could only print the fields it held.
The Go-side anomaly classifier stays, and its output becomes friction notes — a richer `loom-halt` note and a new `loom-crash-resume` note — that the Tier 2 reflection agent reads and files only when they show a lyx problem.
`lyx loom start` vouches for the resume it spawns, so a deliberate resume or a migration between machines never reads as a crash.
A test-enforced `PATTERN-agent-filed-issues` keeps `selfreportengine.CreateIssue` reachable only from `lyx selfreport create`.

## Card Index

1 — remove-tier1-filing — delete loom's GitHub filer, its marker, its seam and the issue rendering
2 — retire-selfreport-key — drop the `selfreport` key; the entry observation follows Tier 2's on/off
3 — rename-handoff-voucher — rename the step clean-handoff marker and the self-report files
4 — anomaly-friction-notes — halt notes carry the anomaly kind and history; entry writes a crash-resume note
5 — start-vouches — `lyx loom start` writes the handoff voucher right before it spawns a driver
6 — agent-filed-issues-docs — PATTERN entry with its scan test, reflection stencil, friction docs and overview

## Shared Decisions

- Tier 2 on is `c.frictionDir != ""`, `internal/friction`'s single "off" signal; every former `cfg.Selfreport` gate reads it.
- Note writes go through `friction.NotePath`, are warned on failure, and never change a verb's exit or envelope.
- Tests for removed code are deleted, not adapted; the classifier and note rendering stay untagged table tests.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go test ./...
go test -tags integration ./internal/loomcli/... ./internal/loomengine/... ./internal/webstercli/... ./internal/orchcli/... ./internal/friction/... ./internal/shedverbs/... ./cmd/lyx/... ./contracts/...
go vet -tags smoke ./internal/loomcli/... ./internal/orchcli/...
```
