```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens on F1, the only finding whose premise is a set (the auxv wiring sites).

No findings.

The one search was a grep over the repo's Go files for `unix.Auxv`, `clockTicksFromAuxv` and `auxvClockTicksKey`. It found one production site that reads the real vector, `proctree_linux.go:36`, and the new test calls `unix.Auxv()` itself, so the fix covers that site. The other hits are the pure helper and its synthetic-vector table test. The review named no other site.

Unchecked: no other finding, since F1 is the only one.

Tool calls:
1. grep over `*.go` for `unix.Auxv`, `clockTicksFromAuxv` and `auxvClockTicksKey`, to rerun F1's search for sites and members. My context lacked a repo-wide listing.
