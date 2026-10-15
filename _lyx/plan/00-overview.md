---
format: 5
approved: false
language: go
---

# Plan: Webster's Master reads no code

Webster's Master reads only what every fork shares and what stays stable for the whole run: the plan's `00-overview.md` and, when PATTERN is active, the `pattern/` background files its constraints section points at.
It reads no card, source, test or doc file, because forks rewrite the code and a read a later fork inherits would be stale.
Each fork, the recovery strand and the verify-gate fixer read the code their work needs themselves.
The change is wording only: the master, fork-prefix, recovery-prefix and verify-fix stencils, the three live FRESH-READ fragments, the docs that describe the Master's reads, and the claim and render pins on that wording.

## Card Index

1 — master-read-set — the Master template names its read set, forbids any other read, and the docs say the Master never reads code
2 — fork-no-code — the fork prefix and the three live FRESH-READ fragments say no code reaches the fork from Master
3 — recovery-orientation — the recovery prefix orients from the overview and the rules and reads code per card
4 — verify-fix-no-code — the verify-gate fixer is told its inherited context holds no code

## Shared Decisions

- `webster-fragment-fresh-read-pack-inline.md` stays byte-identical: it is the frozen layout baseline, and `inlineFreshReadBody` in `internal/websterengine/template_test.go` keeps pinning it.
- No Go behavior changes, no stencil marker is added or removed, and no fragment is added (`PATTERN-stencil-ownership`).
- Claim rows assert short phrases that must hold, never whole paragraphs, and every updated pin extends an existing row of `contracts/stencils/claims_test.go` or `internal/websterengine/template_test.go`; no new top-level test is added.
- No card implements decision D8: the post-landing `tools/forkaudit` re-measurement reaches the change description through the Describe row.

## verify:

```bash
go build ./...
go vet -tags integration ./contracts/stencils/... ./internal/websterengine/...
go vet -tags tmux ./contracts/stencils/... ./internal/websterengine/...
go vet -tags llm ./contracts/stencils/... ./internal/websterengine/...
go test ./contracts/stencils/... ./internal/websterengine/...
go test -tags integration ./contracts/stencils/... ./internal/websterengine/...
```
