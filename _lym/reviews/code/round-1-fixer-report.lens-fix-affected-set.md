```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the finding sets (F4 stale refuser comments, F5/F3 documentation sites) and check the fix covers every member.

**Findings**

- **LOW** — `internal/fabricengine/doc.go:512`. The doc still says `clone --reset` removes the hub "only after the probe has checked the warp binding, the shortname and the old-order guard". The fix added a staged `--subpath` check before removal (`checkSubpathBeforeReset`), and the `CloneHub` doc comment was updated to say so. This sibling statement was not. The search was `grep "removes the existing hub|--reset"` over non-test Go files, and it returned `doc.go:512` with the old wording.

**Unchecked**

- F1: the finding names one test function, so there is no wider set to search.
- F3: its premise is two refusals, and the fix added rows for both. The sentinel-file proof of survival and the extra subpath rows go beyond the review.

**Checked, no gap**

- F4: the reworded comments and the review's other named sites are covered. The grep for `resetHub` and `clone.go:<n>` line references found nothing stale beyond them.
  - `livestate_mutationoracle_test.go:194,318` say `resetHub`'s whole-hub teardown, which is still accurate.
  - `clone_reset_guard_test.go` drives `resetHub` directly, so it is accurate.
  - `livestate_doc_test.go:183` is a past-run mutation record. The review already rejected it.

**Tool calls**
1. A grep for `resetHub`, `clone.go:<n>` and `pre-flight` across `internal`, `cmd` and `docs`, outside `clone.go`. Reason: rerun the F4 set beyond the files the review named.
2. A grep for `--reset` and removal-order wording in non-test Go files. Reason: find docs and help text that restate the removal order after the `--subpath` change.
3. A grep of the saved output of call 1 for `resetHub` and `clone.go:<n>`. Reason: the output was truncated.
