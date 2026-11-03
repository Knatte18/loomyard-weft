```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out as proposed. Those are F1 (the extra `TestSeedDiscussedRecord_Real` rows) and F3 (the glossary edits).

no findings

- F1: I could not build a case where the proposal is wrong.
  - The existing-record row seeds a different record and a support log before the seam runs. It asserts the record is overwritten, the log is kept byte for byte, and both are committed. This follows from `seedDiscussedRecord` writing the record unconditionally and writing the log only when `os.Stat` reports not-exist.
  - The missing-file row asserts false and nil. The tier-1 run passed it, so `DecisionRecord` reports not-found when the file is gone.
  - The blocked-directory row puts a regular file at `record.dir`. `MkdirAll` then returns a `PathError` that carries that path, so the "names the directory" assertion holds.
  - Each row uses its own slug and pair, so the rows share no state.
- F3: the rewritten motif row says a board task may name `motif` and that it runs Discussion alone. `internal/battencli/wire.go` (lines 532 and 538) admits `motif` as a task recipe, and `contracts/recipes/motif-recipe.yaml` exists. The edited lines now agree with the unchanged line 28 and the cartoon row, which still point at the later Plan block.

Tool calls beyond the context: none.
