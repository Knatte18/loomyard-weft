---
format: 5
approved: true
language: go
---

# Plan: Webster fork deviations list mixes in its own target files

Webster's digest `deviations` come from `planglyph.ScopeGuard`, which today counts a touched symbol in scope only by exact match against the batch's target glyphs, so a card that targets a file or package reports every member it edits there.
This plan exports planparser's member classification, widens `ScopeGuard` so file and package targets cover the symbols in their extent, states one deviation rule in the implementer body the fork and recovery prompts compose, and points the plan spec's deviation paragraph at `ScopeGuard`.

## Card Index

1 — planparser-ref-member — export `refMember` as `RefMember`, nil plan reading as the default language
2 — scope-guard-file-package-cover — `ScopeGuard` treats file and package targets as covering the symbols in their extent
3 — implementer-deviation-rule — one deviation rule in the implementer body's per-card step 2
4 — plan-spec-scope-guard-pointer — the plan spec's deviation paragraph points at `planglyph.ScopeGuard`

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
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
