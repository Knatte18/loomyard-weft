```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. I judged what the amendment forces outside its own lines. The amendment is the 14 additions under `docs/research/` from 9c167c500 and the one allowlist entry in 015a8b14e.

**Per change, the set found:**

1. **Research record, `.md` files under `docs/research/codex-astra-solo-bench-2026-10-25/`.**
   - The only reader the diff can affect is `TestEnforcement_MarkdownLinks`, which scans `docs/` through `docsLinkScan` (`internal/lyxcwd/docslink_test.go:~335-360`). The record's one broken link, `pattern/PATTERN-owner-machine.md` in `data/reviews/P4/opus-medium-r1.md:20`, is covered by the new allowlist entry. The report says nothing about the other record files. I did not check their links.
   - Covered.
2. **Allowlist entry, `docsLinkAllowlist`.**
   - The key is `(file, target)` with the file path in full, so it matches exactly one link in one file. A broken link elsewhere, including in the research record's other files, is still reported.
   - The entry self-expires. A stale entry fails the scan through `allowlist.Stale()`, so the entry must be deleted once the link is fixed or the file renamed.
   - Covered, and not wider than that one record.
   - The `docsLinkAllowlist` doc comment still says "known-broken links this task leaves for other tasks to fix". The entry's `Why` sits close to that purpose. I raise no finding.
3. **D3 `TestEnforcement_LocationLiterals`.**
   - It walks Go ASTs (`internal/lyxcwd/enforcement_test.go:922-945`). The amendment adds no `.go` file beyond `docslink_test.go`, which is a test file. The 14 added files are `.md`, `.sh`, `.tsv` and `.txt`.
   - Not affected.
4. **Other consumers of `docs/`.**
   - My grep found tests that mention a `"docs"` path (`cmd/lyx` is absent; `boardengine/render_test.go`, `runprofile`, `planparser` and others). The directory name in them is incidental to the research record, and none walks `docs/research`. The grep did not prove that.
   - Docs rules (`docs/overview.md`, `doc.go`, PATTERN) are not forced by an additions-only research record.
   - Not affected.

**Findings: none.**

Tool calls (2 of 6 allowed):
1. `grep -rln '"docs"' --include=*_test.go internal cmd`, to find tests that read `docs/`. My context lacked this.
2. `grep -n 'Location' internal/lyxcwd/*_test.go`, to confirm that D3 is an AST scan over Go files. My context lacked it.
