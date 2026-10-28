```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes to cards 3–9 still agree with the surrounding cards, decisions and cross-references.

1. **MEDIUM.** Card 6, Intent: the new out-of-module marker paragraph against the unchanged sentence above it, and against D5.
   - **Summary:** the fix lets a marker whose unit's first segment names no root directory pass without a declaration check. Its twin sentence still says the scan checks that a marker "exists and names a struct". D5 still says the scan "refuses a marker whose glyph does not name a struct". So the new bound contradicts the card's own sentence and narrows D5. No decision-record entry licenses the narrowing, which is the defect F8 raised against card 4.
   - **Evidence:** `_lym/plan/06-seam-marker-scan.md`, the line "For a test reflecting directly, the scan checks that a marker exists and names a struct" next to the new "is not checked against a declaration" paragraph; `_lym/discussion/decision-record.md:95` and `:99`.

2. **LOW.** Card 4, Intent: "`internal/gitrepo/doc.go` names it beside `SubdirsAtRevision` among the go-git local reads."
   - **Summary:** the `SubdirsAtRevision` bullet in that doc ends with "All three are go-git local reads." Adding `TrackedRootEntries` there leaves the count wrong unless the card also rewrites that sentence, and the card does not ask for it.
   - **Evidence:** `internal/gitrepo/doc.go`, lines 70–74 as quoted earlier in context: TagRefs, BranchRefs, Log and SubdirsAtRevision, followed by "All three are go-git local reads."

3. **LOW.** Card 4, the Create group and the `TestTrackedRootEntries` paragraph.
   - **Summary:** the new tier1 test in `internal/gitrepo/worktree_test.go` needs a repository with entries that are committed, staged only and untracked. Neither the card's Uses group nor its Intent names the gitrepo_test fixture helpers it would use, `newRepo`, `writeFile` and `commitAll`, which that file already reuses. Every other test the card adds names its fixture source.
   - **Evidence:** the `worktree_test.go` header quoted in context ("reusing gitrepo_test.go's newRepo/writeFile/ commitAll fixture helpers"); card 4's Uses lists only `gitkit#CopyRepo` and `gitkit#Git`, which belong to the loomshed test.

Tool calls:
- grep of `_lym/discussion/decision-record.md` for the D3, D4 and D5 wording, and of the plan cards for leftover HEAD, no-commit and empty-list text: my context held no decision-record text beyond the review's quotes.
