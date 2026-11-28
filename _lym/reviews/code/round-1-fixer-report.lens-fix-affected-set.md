```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran the searches for the findings whose premise is a set of sites.

**Unchecked findings:** none of the set-premised ones (F1, F3, F6, F7, F8, plus F9's support-log wording). I treated F2, F4, F5, F10, F11 and F12 as single-site and did not re-search them.

1. **MEDIUM: F1's change leaves a production writer of the seed param as a bare literal.**
   - `internal/battencli/wire.go:639`, in `childSeedParams`, still has `return map[string]string{"parent": origin.ParentBranch}, nil`.
   - This writes the same seed param that the child's bootstrap and Record-Seed read through `loomshed.SeedParentParam`. Its doc comment says a disagreement with the bootstrap makes the bootstrap refuse.
   - `wire.go` already imports `internal/loomshed` (line 45), so using the constant needs no new import.
   - `PATTERN-code-conventions` says: "one named constant, declared once and referenced everywhere, never a repeated literal".
   - The review named only three sites. The line predates this diff, but F1's premise (one name, defined once) stays unmet.

2. **LOW: F1's change leaves the twin way-forward text spelling the key by hand.**
   - `disagreeingSeedRefusal` (`internal/loomcli/sharedbootstrap.go:254`) formats `--param parent=%s`.
   - `recordSeedStuckReason` renders the same command with `SeedParentParam`.
   - `internal/loomcli/arm.go:95` also has `--param parent=<parent-branch>` inside a message string.

3. **LOW: F7's change leaves two test comments saying `tapestry-recipe.yaml` holds the rows.**
   - `internal/loomrecipe/sequence_test.go:31` says "a reordering of the rows in contracts/recipes/tapestry-recipe.yaml therefore fails the test".
   - `internal/loomcli/smoke_test.go:18` says "The producer table in contracts/recipes/tapestry-recipe.yaml backs every row with a real producer".
   - That file now holds only the Motif and Jacquard block rows.

The F3, F6 and F8 searches turned up no further stale statement.

Tool calls (6):
- Grep for the parent seed-param literals under `internal` and `cmd`: my context held only the review's three sites and an earlier, narrower grep.
- `git grep` for tapestry row and header references: my context held only the filename hits, not prose references.
- `git grep` for the old loom-contract, block and support-log wording: it failed on a bad pathspec and returned nothing.
- Grep of `jacquard-recipe.yaml` and the other recipes for cross-references: my context showed only the first 290 lines of jacquard.
- The same old-wording `git grep`, with a corrected pathspec: not in context.
- A read of `battencli/wire.go`, its `loomshed` import, `disagreeingSeedRefusal` and the Named constants pattern: none of these were in context.
