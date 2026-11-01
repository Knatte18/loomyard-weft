```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's set search (F1 wake-path done-flag branches, F2 `samplerStrand` call sites) and check the fix covers every member.

No findings.

- F1: the wake path reaches `judgeDone` only at `sampleression.go:372`. Its branches are dirty, damaged record and clean Done. The two new rows cover dirty and damaged on the wake path, and the clean-Done branch already has the "flag written during the wait" row. The error branches (`DirtyPaths`, `CheckRecord`, `stopLive` failing) have no wake-path row. The review did not name them, and the seams are not driven to fail in the world fake, so I leave them unchecked.
- F2: `deps.samplerStrand` has one non-test call site, `decision.go:651`, and the fix changes it. The new success row covers the parent with a failing `samplerStrand`. The existing "sampler session check failure" row still refuses a non-parent caller (empty parent name).

Unchecked: none beyond the wake-path error branches noted under F1.

Tool calls:
1. `grep -rn "samplerStrand("` over non-test Go files, to list every caller of the F2 seam; my context lacked a repository-wide caller search.
