# Card 1 — edit-directive-wording

**Edit:**
- `contracts/stencils/edit/edit-directive.md`
- `internal/editdirective/doc.go`
- `contracts/stencils#editRulePhrases`
- `internal/editdirective#TestDirective_RendersTheRuleWithoutBannerOrMarkers`
- `contracts/stencils#TestParentDirectiveScans_SyntheticRows`

**Intent:**
D1's wording: the directive bans the in-place forms, allows reading with `sed -n`, and names a scratch place.

`contracts/stencils/edit/edit-directive.md` keeps its first clause, "Change files only with Edit or Write, never with a script that rewrites them", and the one-Edit-per-site and `replace_all` sentence.
The script ban names each rewriting form directly after its own "never", so `PATTERN-no-denied-recovery`'s exemption marker admits it once card 5 lists the forms: never `sed -i`, never `perl -i`, never `awk`, and never `python` or `node` in an inline heredoc.
A new sentence says reading with `sed -n`, `grep` or `cat` is fine.
A new sentence says scratch files go under `.scratch/` in the working directory, never a system temp directory, and are removed by their literal path.
The leading comment stays as it is, and the file stays marker-free.

`internal/editdirective/doc.go`'s "What it says" paragraph restates the rule summary to match: Edit or Write only, never `sed -i`, `perl -i`, `awk` or an inline interpreter heredoc, reading with `sed -n` allowed, scratch under `.scratch/`.

In `editRulePhrases`, the entry "`sed`" becomes "`sed -i`", so `TestStencils_NoEditRuleRestatement` still finds the directive's statement and no stencil that mentions reading with `sed`.
Its doc comment is unchanged.

`TestDirective_RendersTheRuleWithoutBannerOrMarkers` asserts the rendered text holds "never `sed -i`", "never `perl -i`", "never `awk`", "never `python`", "`sed -n`" and "`.scratch/`", beside its existing "Edit or Write" and "replace_all" checks; the bare "sed" check goes, since the new ones cover it.
`TestParentDirectiveScans_SyntheticRows`'s "body restating the edit rule" subtest gains two cases of `editRuleSentences`: "Fix it with `sed -i`." yields one sentence, and "Read the range with `sed -n`." yields none.

**ImpactSummary:** Rewords the one edit-directive stencil every spawned role renders, its package doc and one phrase of the single-statement scan; no Go behavior beyond the scan's phrase list changes.
