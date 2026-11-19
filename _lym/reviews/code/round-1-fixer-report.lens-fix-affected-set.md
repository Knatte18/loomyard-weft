```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran the set searches for F1, F2, F3, F4 and F6. No findings.

Findings I left unchecked: F5 (the decision ruling), F7 and F8 (prose edits), F9 to F12 (single-site test additions) and F13. Their premises are not sets.

- **F1:** the only remaining `No test seam` literals are the `NoTestSeamTitle` declaration, the test assertions, and prose in `darn-recipe.yaml:139`, `darn-template-write.md:112` and `docs/overview.md:1105`. The prose files cannot reference a Go constant. Every Go declaration of the title and the `bug` label that the diff added now lives in `gateslot`.
- **F6:** every non-test file calling `verifytree.Verify` outside `internal/verifytree` is in `planVerifySites`, and that list now includes `internal/loomshed/redfirstgate.go`. The other hits are `doc.go` files.
- **F2:** the new exit code `RedFirstUnjudgedExit` is handled in `redfirst.go`, the `redFirstFailure` switch, the gatecli, gateslot and loomshed docs, the verb's `Long` help and `docs/overview.md`. The remaining "prints a note naming the scoper, with exit 0" lines at `gatecli/doc.go:751` and `:763` belong to the other verbs (redundancy, mutate), not red-first.
- **F4:** `baseOverlay` has two callers, the verb and its test, and both use the new three-value signature.
- **F3:** `FailedTests` is still used by `loomshed/verifygate.go:248` and `landingshed/verifygate.go:320`, unchanged. Only `judgeRedFirst` uses `NamedFailedTests`. The old mixed-report bound and the old non-Go-files bound no longer appear in any file.

Tool calls: 1 (a single `Bash` batch of greps). My context held the changed files but not repo-wide search results for these sets.
