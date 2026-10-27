```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set — find what the diff forces outside its touched lines.

**Affected-set list**

- `docsLinkIsResearchData` (new, one caller). Its only caller is `docsLinkScan` (`docslink_test.go:380`). Covered.
- `docsLinkScan` skip. The other callers are `TestEnforcement_MarkdownLinks` (line 420) and the subtests (lines 455–610). Their fixtures are all rooted at `.` or `docs` and none puts files under `docs/research/*/data/`, so none changes behavior. Covered.
- Allowlist entry removal, `docsLinkAllowlist`.
  - The only reader is line 420.
  - Subtests that exercise the allowlist (lines 518–556) use their own local `allow` slices, not this variable, so they still hold.
  - The `Why`/`Key` text appears nowhere else. A match in `contracts/stencils/citation_enforcement_test.go:41` is a comment about the allowlist shape, not the entry. Covered.
  - The variable's doc comment (lines 405–408) still reads "known-broken links this task leaves for other tasks", which is odd for an empty list. The text is unchanged, so this is not a finding.
- Docs and PATTERN.
  - `grep` for "Markdown Link Integrity" and "docs/research" in `PATTERN.md`, `pattern/`, `docs/overview.md` and `internal/lymcwd/doc.go` returned nothing, so the record's claim that no PATTERN text states the guard's scope holds. The only other `.md` hit is `docs/research/tsa-quarry-2026-10-24/report.md`, which is verbatim research data and out of scope.
  - The file header and the guard's doc comment both carry the exclusion. Covered.
- Resolution into `data/` files. The skip sits only at the source side of the walk, so links from scanned files into `data/` (anchors included) still resolve unchanged. Covered.
- Stencils and fixtures. The diff changes no stencil or fixture. Not applicable.

**Findings:** none.

**Tool calls (1)**
1. `grep` for `docsLinkScan`, `docsLinkAllowlist` and `docsLinkIsResearchData` in `*.go`, and for "Markdown Link Integrity" or `docslink_test` in `*.md` outside `docs/research`. My context only showed the diff and an earlier, narrower grep.
