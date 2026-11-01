No findings.

Task: with the lens fix-affected-set, I re-ran the search behind F4. F4 is the only finding whose premise is a set, here the stencils that tell an agent to re-run `await-forks`.

- **F4 (stencils):** The search over `*.md`, `*.go`, `*.yaml` and `*.tmpl` finds exactly three stencils that carry the re-run instruction: `burler-cluster-fan.md`, `burler-fix-self-check.md` and `twill-directive.md`. The fix edited all three, with the same bound and fallback text: up to five `ready: false` answers in a row, and a sixth ends the turn.
- **Other doc mentions:** `internal/twilldirective/doc.go:9`, `internal/burlerengine/doc.go:82` and `:277`, and `docs/overview.md:419` describe the wait without a re-run rule. They are not stale, and the bound is deliberately kept in the stencil text only.
- **Research docs:** The two files under `docs/research/` record a benchmark run. They are measurement reports, outside the set, and are not stale.
- **Unchecked:** F1 is a single-site fix. F2, F3 and F5 are coverage gaps, not set findings. F6 is one error string. I did not search these.

Tool calls (1):
1. A repo-wide `grep` for `ready: false`, `ready false`, the re-run phrase and `await-forks`. My context held the three stencil edits but not the other files that mention the re-run rule.
