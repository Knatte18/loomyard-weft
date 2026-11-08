```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran F1's search for dropped tests that assert archive tags. The fix covers all seven named tests. It left four unnamed members of the same set.

Search: `git diff f2ac2bf7 HEAD -- internal/fabricengine`, filtered to removed test functions whose removed lines match `archiveTagsAt|ArchiveTag|OriginArchiveTag|SetAsideTag|"archive/|refs/tags|tagTargetAt`. It returned these, none of them among the seven:

- **MEDIUM** `TestRemove_DirtyRefusalLeavesPairIntact`: its removed line `assertNoArchiveTag(t, h.RecordsBare, weftRoot)` is the same assertion as the restored `TestRemove_OutOfPathspecSiblingDirtRefusesBeforeAnyMutation`, "no origin tag after a refused Remove". Unless a kept test pins it, F1's premise still holds for this test. I did not confirm whether `TestRemoveRefusal_ProbeLeavesPairUntouched` or another kept test covers it. That test only calls the read-only `RemoveRefusal`, not `Remove`.
- **MEDIUM** `TestCleanup_PrefixSetOverBareSlugOrphan`: the removed lines `strings.HasPrefix(entry.ArchiveTag, fabricengine.ArchiveTagPrefix+slug+"/")` assert the archive tag's name. That is a tag assertion in the category the record says to keep.
- **LOW** `TestRemove_NoOriginSkipsArchiveAndCompletes` and `TestCleanup_NoOriginDeletesAndReportsArchiveSkip`: both assert that no tag exists and a skip reason is reported. This is a negative tag assertion, which the review did not flag. The fixer chose not to restore the first. Say so in the report.
- **LOW** `TestRemove_ReturnsTheRemoteStepFailureOverAKeptDirectory`: its removed `SetArchiveTagPushHookForTest` hook fails the tag push. I could not see whether it asserts a tag. It is a candidate only.

The seven restored tests are present in the changed files, each with `testtiming:keep`. I did not re-check them.

Tool calls:
1. `git diff f2ac2bf7 HEAD` with grep/awk, a first attempt that was cut off at `head -80`. My context lacked the full set of dropped tests that assert tags.
2. The same diff with a narrower filter, to list every dropped test that asserts a tag. My context held only the review's seven.
