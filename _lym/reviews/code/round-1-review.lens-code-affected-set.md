```yaml
lens: code-affected-set
outcome: reported
```

**Task:** list what the `clone --reset` reorder forces outside the changed lines, and which of it the diff leaves wrong.

## Set per change

| Change | Set found | Covered? |
|---|---|---|
| `resetHub`'s stat and `looksLikeHub` pre-flight moved into `checkResetTarget`, called by `CloneHub`'s two-argument offline stage and by `resetHub`. | `clone_reset_guard_test.go` calls `resetHub` directly and still holds. `export_test.go:146` (`LooksLikeHubForTest`) and `destroy.go:364` and `destroy.go:629` use `looksLikeHub`, which is unchanged. | Covered. |
| The two-argument non-hub refusal is now raised by `checkResetTarget` in `CloneHub`, not by `resetHub`. | The comments below name `resetHub`'s own pre-flight, with `clone.go:573-577` line numbers, as the refuser: `livestate_refusal_selftest_test.go:310` and `:321`, `livestate_verbs_test.go:51` and `:992`, `livestate_matrix_test.go:313`, `livestate_doc_test.go:183`, and `livestate_mutationoracle_test.go:194`. | Not covered. These now describe the wrong function. |
| `--reset` removal runs after the probe in both forms. | `fabricengine/doc.go:512` was updated. `fabriccli/clone.go:195`, `fabric.go:70` and `:97`, `fabriccli/doc.go:17` and `docs/getting-started.md:24` stay true. `warpbinding_clone_integration_test.go:719-724` stays true for the non-reset case. | Covered. |
| The mutation record's `path_removed "."` entry now follows the probe. | The recorder is created before the probe, as before. The `TestCloneHubReset` oracle compares the record to a manifest diff, which the reorder does not change. | Covered. |
| PATTERN entries. | `PATTERN-fabric-boundary` (destruction chokepoint): `destroy.go` still holds the only destructive primitive, and `checkResetTarget` only reads. `PATTERN-code-conventions` (doc.go): the new `doc.go` sentence documents package behaviour, not a rule for other code. The PATTERN diff is empty and the rubric declares no PATTERN changes. | Covered. |
| Commit messages. | There is no `Pattern-Departure:` trailer and none is needed. The commit message matches the diff. | Covered. |

## Findings

**1. LOW, consistency: comments name `resetHub` as the refuser of a non-hub target, which is no longer true in the two-argument form.**
- Where:
  - `internal/fabricengine/livestate_refusal_selftest_test.go:310` and `:321`;
  - `internal/fabricengine/livestate_verbs_test.go:51` and `:992`;
  - `internal/fabricengine/livestate_matrix_test.go:313`.
- Detail:
  - `livestate_refusal_selftest_test.go:310` says "resetHub's own looksLikeHub pre-flight ... refuses before CloneHub ever touches the network". The test passes `WarpURL`, so it is the two-argument form. The refuser is now `checkResetTarget`, called from `CloneHub` at `clone.go:178`.
  - `livestate_verbs_test.go:51` makes the same claim and adds that the refusal comes "BEFORE the pathRequest at :579 is built". `resetHub` does not run at all in this form when the target is a non-hub.
- Evidence: `clone.go:177-180`, `clone.go:623-627` and `clone.go:644-664`.
- Fix: reword these to name `checkResetTarget`, or "the offline pre-flight".

**2. LOW, consistency: the `livestate_doc_test.go:183` mutation-test record names the neutered guard as `resetHub`'s own pre-flight.**
- It says "Neutered clone.go's resetHub own pre-flight (the `if !looksLikeHub(hubPath)` guard)". That guard now lives in `checkResetTarget` (`clone.go:658`).
- The record is a measurement report of one past run, so it is exempt from the perishable-detail rule. It is worth a note only if the file is meant to stay a live pointer.
- This is probably not worth raising on its own.

## Tool calls
1. `grep` across the repo for comments and docs describing reset ordering, or `resetHub`'s pre-flight. My context lacked the stale-comment set outside `clone.go`.
2. `grep` of the docs and README for `clone --reset`, plus `sed` of `fabric.go:60-75`. My context lacked the operator-facing help and docs.
3. `sed` of `livestate_doc_test.go:180-190` and `livestate_refusal_selftest_test.go:305-324`. My context lacked the text of these two comments.
