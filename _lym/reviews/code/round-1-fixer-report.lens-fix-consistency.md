```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens on the F1–F5 change set.

**Finding 1 — LOW.** `internal/fabricengine/doc.go:943`, the package doc paragraph on `failRejection`.
- The summary says only "a failed fetch leaves origin unknown and is an `other` failure too."
- F1 added a second `other` case in `failRejection`: a branch with no upstream, where `UpstreamSHA` fails after a successful fetch. The paragraph and the `failRejection` doc comment now disagree, because the comment in `pushclassified.go` was updated and `doc.go` was not.
- Proposed change: add "or a branch with no upstream to compare with" to the `doc.go:943` sentence.

**Other cross-references, all consistent:**
- The `pushClassified` doc comment (`pushclassified.go:92`) says "`other` otherwise", which still holds.
- `docs/benchmarks/test-redundancy.md:713` names the old test `…Unwrapped`. It is a measurement report, so leaving it stale is correct under the prose skill's exception.
- The F5 header and rename in `pushanchored_integration_test.go` agree with each other, and no other reference to the old name remains.
- The new `RecordsWithNoUpstream` row and the fetch-failure subtest match the docs and the code.

**Calls:**
1. `grep` for the old test name and for `failRejection`, `no upstream` and fetch wording in `doc.go`: my context had no view of the other files and docs that cite either.
2. `sed -n 939,945p internal/fabricengine/doc.go`: to read the full `failRejection` paragraph, which my context lacked.
