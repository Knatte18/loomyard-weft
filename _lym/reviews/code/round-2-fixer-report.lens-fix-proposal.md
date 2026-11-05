```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F1 proposal, which was to parse the base only when `haveBase` is true. I fixed that finding as proposed.

no findings

- **Skipped parse, caller case:** `configsync.go:1046` passes `templateBasePresent && !fileAbsent`. When it is false, the base is either absent or about to be rewritten by the same pass. Skipping the parse changes no result there.
- **Skipped parse, followed output:** `baseRoot` stays nil, which is what the removed `if !haveBase { baseRoot = nil }` produced, so the follow output is the same.
- **Parse when `haveBase` is true:** the code still parses and still wraps the error in `ErrInvalidBase`, so the `configsync.go:1047` branch is unaffected.
- **New test row:** it passes the base `"- not\n- a mapping\n"` with `haveBase` false. With the old code the base parse would fail and the row's `t.Fatalf` on a non-nil error would fire, so the row fails with the defect in place.
- **Doc edits:** the wording "so `base` is not read" in the `FollowTemplate` comment and in `docs/shared-libs/yamlengine.md` matches the code.

Tool calls made: none; my context already held every file I needed.
