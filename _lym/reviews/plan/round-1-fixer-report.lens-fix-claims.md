```yaml
lens: fix-claims
outcome: reported
```

fix-claims: checking every claim the fix added or rewrote against the code at HEAD.

**1 finding (LOW)**

- **LOW** — Card 4, Intent: "`internal/gitrepo/doc.go` names it beside `SubdirsAtRevision` among the go-git local reads."
  - **Issue:** if the new read goes where the card says, the doc's own count goes stale, and the card doesn't say to rewrite it.
  - **Evidence:** `internal/gitrepo/doc.go` (around line 73) puts `SubdirsAtRevision` in a group of three reads (`TagRefs`/`BranchRefs`, `Log`, `SubdirsAtRevision`) and closes that group with "All three are go-git local reads."
  - Adding `TrackedRootEntries` there makes "All three" wrong.
  - The Intent should either say the sentence becomes a count-free form, or give `TrackedRootEntries` its own bullet.

**Claims that hold**

- **Card 3:** `resolveTargets`' doc comment does hold a consumer list (`internal/planglyph/repo.go:51`).
- **Card 4:**
  - The `wireLightweight` comment does say the validate verbs read only `DecisionRecordPath`/`SupportLogPath`/`AnchorPath`/`WorktreeRoot`.
  - "An absent index is an empty result" holds: go-git's `IndexStorage.Index()` returns an empty `Version: 2` index on `os.IsNotExist`.
  - `internal/gitrepo/worktree_test.go` is an existing tier1 file in package `gitrepo_test`.
- **Card 8:**
  - `glyph-not-found` for the `internal/c_test#TestDeps` and `internal/c_test#` Uses rows holds.
  - `statusFindings` indexes Uses refs (`targetCards`, `internal/planglyph/resolve.go:16-40`).
  - quarry answers `internal/loomshed_test#…` and `internal/loomshed_test#` as `not_found` (checked in the parent's context).
- **Card 9:** "D8 holds only a card touching non-test Go code" matches the decision record's D8 ("touch non-test Go code of a package directory").
- **Card 6 and card 5:** the out-of-module marker rule is consistent between them.
  - A `time#Time` marker gets `GlyphMissing` from card 8's `ResolveGlyphs`, which only yields no declaring files and no error, so no claim conflicts.

**Tool calls**

- Read decision-record lines 125–135 and `statusFindings`' unit branch: I needed D8's exact wording and whether the resolve pass covers Uses refs.
- Read go-git's `IndexStorage.Index`: I needed its behaviour on an absent index.
- Read `internal/planglyph/resolve.go` lines 70–100 and its Uses indexing: I needed to confirm a Uses ref reaches `glyph-not-found`.
