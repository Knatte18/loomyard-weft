```yaml
lens: fix-affected-set
outcome: reported
```

Fix-affected-set lens: I reran each finding's own search against the changed files. The sets are covered, apart from one stale comment.

Unchecked: F1 was disputed by the fixer, so I did not search it. F2, F3 and F4's gate.yaml-example set needed no further search beyond the one I ran.

**LOW — stale comment on `greenWholeCommitsKept`**
- Changed text: `internal/gatecli/green.go:33`, the doc comment "how many of a tier fingerprint's newest whole-green commits a write keeps beside the current tree's entries".
- Summary: F6 changed the budget to count only whole entries with empty args. The constant's own comment, and `internal/gatecli/doc.go:126` ("each fingerprint's 20 newest whole-green commits"), do not say so. `green.go:95` was updated.
- Evidence: `green.go:139` now skips `len(held.Args) > 0`, and the grep shows `green.go:34` and `doc.go:126` unchanged. A reader of the constant or of the doc would not learn that filtered whole entries are excluded and survive only for the current tree.

**Searches that found nothing missed**
- **F5, other callers of `newestWholeAncestor`:** the only callers are `scope.go:241` and `green_test.go:241`. The one-argument seam appears nowhere else. The `isAncestor*` helpers in `loomshed`, `websterengine` and `fabricengine` are unrelated functions.
- **F6, other places that count whole entries:** the only sites are `green.go:139` and `green.go:~150`, both edited.
- **F4, other old tag spellings in the gate.yaml examples:** `internal/gatecli/doc.go` has no remaining `-tags integration|tmux|llm`. Remaining hits are `internal/gatecli/tiers_test.go:23`, which is a planning fixture that never executes a tier, and benchmark and research docs, which are measurement reports and out of scope.

Tool calls:
1. One Bash call running three greps (F4, F5, F6): my context held the changed files but not a repo-wide search for the finding sets.
