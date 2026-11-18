```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F8 and F9 fixes cover every member of their sets.

No findings.

- **F9 (cgroup path literals):** the only remaining `"/proc/self/cgroup"` and `"/sys/fs/cgroup"` literals in Go code are the declarations in `internal/proc/tasklimit.go:16-17` and `cmd/testtiming/resources.go:68`. The `testtiming` one is a separate command with its own `cgroupRoot` constant that the review never named, so the fix does not reach it. The holder test now uses `proc.SelfCgroupFile` and `proc.CgroupMount`.
- **F8 (keep markers):** `//testtiming:keep` appears in `internal/proc` exactly twice, at `memorycap_linux_test.go:178` and `:283`, directly above the two tests the redundancy report lists as contained. The assist review (`round-2-review.assist-assist.md:54`) names the same two tests and no others. I found no other contained test.

Calls made:
1. `grep` for both cgroup path literals across all Go files, to rerun F9's search. My context held only the two edited files.
2. `grep` for `testtiming:keep` in `internal/proc`, plus `ls` and `grep` of `_lym`, to rerun F8's search and find the redundancy source. My context held no repo-wide marker list.
3. `grep` for "contained" in the assist review, plus `ls` of `_lym/reviews/code` and `grep` of `cgroupRoot` in `cmd/testtiming`, to confirm the contained set has two members and that `cgroupRoot` is not a proc declaration. My context held no list of the redundancy report's entries.
