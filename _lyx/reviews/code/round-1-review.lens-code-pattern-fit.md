```yaml
lens: code-pattern-fit
outcome: reported
```

Lens: code-pattern-fit. I checked each changed file against the PATTERN entries and CLAUDE.md rules that apply to it.

**Rules checked**
- `PATTERN-shell-guard-table`: held. The remaining rows stay in `internal/shellguard`'s one table. Only the two rows and the helpers only they used are removed. `commandPositionPattern`'s comment no longer names the deleted `unquotedPrefixPattern`. The integration test now asserts `sed -i`, `perl -pi`, a redirect and a heredoc append pass every remaining row.
- `PATTERN-parent-directive`: held. The `edit-directive` stencil is still the only statement of the edit rule, and its marker is untouched. The background file's clause now says "the edit rule". The overview line in `PATTERN.md:45` names the edit directive, not the ban, so it stays true. No stray ban wording remains in `PATTERN.md`, `pattern/` or `docs/overview.md`.
- `PATTERN-refusal-way-forward`: held. The closed list and the pattern-shaped clause drop `sed -i` and `perl -i`. `cmd/lyx/norecoverydeny_test.go` is updated to match, with `sed -i` now expected as not denied (count 0). The `editRulePhrases` split-string workaround is gone, since `sed -i` is no longer a denied form.
- Declared PATTERN changes: held. The decision record declares none, and the diff leaves the `PATTERN.md` overview untouched, so there is nothing to land.
- CLAUDE.md "Docs land in the same commit": held. The `claudeengine` doc, `editdirective` doc, `loomcli` doc and `fmtverb.go` header, `docs/overview.md`, `stencils.go` comments and the golden all change in the same range.
- CLAUDE.md "Markdown semantic line breaks": held. The new `edit-directive.md` rule is one sentence per line.
- Commit messages and departures: held. The range carries no `Pattern-Departure:` trailer and no difference from the answer key needs one. The commit messages carry the required Co-Authored-By line.
- `PATTERN-test-speed`, `PATTERN-test-isolation`: held. The integration test change keeps `t.Parallel()` and the table-driven shape.
- Scope against the decision record: `editRulePhrases` loses `replace_all`. The record only says the wording changes, but the new directive no longer states `replace_all`, so the phrase list stays consistent with it. Not a finding.

**Findings**
None.

**Tool calls (1)**
1. `grep -rn` over `PATTERN.md`, `pattern/`, `docs/overview.md` and `CLAUDE.md` for leftover ban wording. My context only had the truncated preview of the earlier grep, which left these files unconfirmed.
