```yaml
lens: fix-affected-set
outcome: reported
```

No findings. The list of findings I left unchecked is empty: I reran each finding's search once, and the fix covers both.

- **F1:** The set is the tests that hold older-template fixtures with `code_reviewer_model: "opus[high]"`. The search found three: `configsync_test.go`, `configcli_integration_test.go:393` and `ensure_integration_test.go:1477`. Each of the other two exercises the same generic reconcile add-and-keep logic, which the fixed `configsync` row now covers. The review's premise targeted only `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`, and both new rows are there.
- **F2:** The set is the contained tests. Only `TestTemplateLadder` (`runprofile_test.go:105`) and `TestTemplateDocsProfile` (`runprofile_test.go:218`) assert the key in `runprofile_test.go`, and both now carry the marker.

Tool calls:
1. Grep of `_test.go` files for `code_reviewer_model`, to enumerate the F1 set: my context held only the three tests the review named.
2. Grep of `internal/runprofile` for `code_reviewer_model` and the test and marker lines, to enumerate the F2 set: my context held only the edited test sections.
