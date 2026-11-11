```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked per finding:
- **F3:** no stale "resumed under either"-style sentence remains in any `.go` or `.md` file outside `_lym` and `.lym`. `internal/adoptcli/doc.go:172` and the new Step done line now agree.
- **F2:** the adopt's reconcile seam is the only `ReconcileOrigin` call in `internal/adoptcli`, at `deps.go:74`, and it uses `reconcileOptions()`. The other `ReconcileOptions{}` sites belong to other packages and are out of scope.
- **F4:** the overview entry at `docs/overview.md:795` now holds the three behaviours. `doc.go:205` already documents the push-lock wait, so no further site needs the sentence.
- **F1, F5 and F6:** these are test-coverage findings with no consumer set. The rows the review named are present: the three untracked-file cases and the two gate refusals, the write-lock row, and the two worker rows.

Left unchecked: F1's premise that `untrackedOnPaths` only reaches the records side, since the review did not ask for it.

Tool calls:
1. Grepped the repo for stale resume-under-either wording, because my context had only the edited lines.
2. Grepped `ReconcileOptions{` in non-test sources, to find other reconcile seams the F2 fix might also need to cover.
3. Grepped `ReconcileOrigin(` in `internal/adoptcli`, to confirm the adopt has one call site.
4. Grepped the docs for pause and push-lock mentions of the adopt, to find any twin of the F4 sentences.
