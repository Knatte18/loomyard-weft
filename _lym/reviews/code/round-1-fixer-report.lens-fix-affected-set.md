```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F3, F2 and F5 and check the fix covers every member.

No findings.

- **F3 (go:embed targets):** `internal/boardengine/board.html` is the only embed target the review named, and `.gitattributes` now pins it. Many other embed targets are still unpinned, for example `contracts/stencils/*` and most `template.yaml` files. They were unpinned before this change and the review did not name them, so I did not count them as findings.
- **F2 (poll and `waitForChange`):**
  - `waitForChange` has one caller, `serveBoard`, and it now passes `hasAfter`.
  - `poll(` in `board.html` has three call sites: the first request `poll(null)`, the next request `poll(result.body.revision)`, and the retry `poll(after)`. The retry keeps whatever `after` was, including null, so a first request that fails retries without `after`.
- **F5 (idle-clock stamp):** `lastAPI` is now written only in `stampAPIRequest`, apart from the initial stamp in `NewServer`. `stampAPIRequest` is called only after `bearerAuthorized` succeeds, for `/api/board` and `/api/write`. No other `apiPathPrefix` or direct `lastAPI` writes remain.
- **F1 (watch loop):** `watchBoard` has one caller, `Run`. It is the only `fswatch.Watch` consumer in `internal/boardserve`.

The remaining findings (F4, F6 to F12) were not rerun as set searches, because their fixes name a single site.

Calls made:
1. `grep go:embed` across the repository, plus `.gitattributes` and tracked html/js files: to enumerate embed targets for F3.
2. `grep waitForChange|apiPathPrefix|poll(|after=` in non-test files: to enumerate callers for F2 and F5.
3. `grep lastAPI|.Now()` in `internal/boardserve`, plus `watchBoard`, `watchEventFloor` and `fswatch.Watch` across non-test files: to check the other stamp and watch sites for F5 and F1.
