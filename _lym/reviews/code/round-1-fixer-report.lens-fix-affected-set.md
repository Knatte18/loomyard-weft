```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

My list was the one set finding, F1, the stale never-died claims. F2 is a single test and has no set. I searched all non-test `.go` and `.md` files outside `_lym/` for the never-died wording and for `errStrandNotTracked`.

- **F1 sites:** the fix covers the three sites the review named: `engine.go:23-29`, the `wait.go:5-7` header and `docs/overview.md:420`. I found no further site that states the claim as a general rule.
- **`wait.go:163-176` (`errStrandNotTracked` doc and message):** it says a strand reed does not track is reed's own bookkeeping going away. It describes the reset causes (`reed down`, `reed.json` loss, worktree rename), not Stop. This is a mechanism-failure rationale, and `wait.go:1242` already records the Stop exception.
- **`tools/sandbox/SANDBOX-SHUTTLE-SUITE.md:179`:** it requires `ok:false`, never `died`, for a blocked run after a reed reset. That scenario involves no `Stop` on the handle, so it still holds.
- **`orchcli/live.go:57` and `orchroute/live.go:17`:** these are about orch liveness, unrelated to shuttle outcomes.
- **`attach.go:565`:** it is about the Attach path with no marked handle, so it is not affected.

Tool calls made:
1. `grep -rniE` across non-test `.go`/`.md` files for the never-died claim wording and `errStrandNotTracked`. This was F1's own search, and my context held no call-site list.
2. Read `wait.go:160-195` to see whether the `errStrandNotTracked` and `errStrandPaneBindingCleared` docs make a general never-died claim. My context held only `wait.go:1-20` and `215-240`.
