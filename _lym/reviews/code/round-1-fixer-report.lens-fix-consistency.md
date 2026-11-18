```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-set for sibling text left stale. Two findings, no BLOCKING.

**LOW — stale "once per record" in code docs after F9's re-arm wording**
- Changed text: `docs/overview.md` (the ask now stays "once per pull request until a send-back clears the ask").
- Stale twins: `internal/landingshed/prreview.go:417` ("It asks once per record") and `internal/landingshed/doc.go:96` ("once per pull request record").
- Evidence: `internal/landingshed/prreview.go:121` sets `r.ParentReviewAsked = ""/false` on a send-back, so the ask re-arms. Both comments still state it as unconditional.

**LOW — Drift docs omit the loosened-check ruling that F9 added to `docs/overview.md`**
- Changed text: the `docs/overview.md` Drift-Verify and Drift-Recheck lines.
- Stale twins: `internal/driftengine/doc.go:550` ("has one judge rule on the expected changes no stored ruling covers") and `internal/driftengine/verify.go:2` (same wording).
- Evidence: those comments describe the judge's rulings as expected changes only, while the judge now also rules on loosened files and can halt Stuck on an uncaused one. This sits in the same package the F4 and F5 tests changed. The reviewer scoped F9 to `docs/overview.md`, so this is a twin of that change.

Checked and consistent:
- No other copy of "Stuck is always a Halt" remains.
- Nothing references the deleted F7 test by name.
- The F10 fixture comment matches the recipe wording.
- The F11 fragment header is self-consistent.

Further tool calls (3 used of 6):
1. Grep for "once per pull request", "always a Halt" and the deleted test's name across the repo; my context did not hold the other copies of that wording.
2. Grep for "expected changes" wording in docs, contracts and internal; my context did not show the Drift doc comments.
3. Read `internal/landingshed/prreview.go:412-422`, `internal/landingshed/doc.go:94-98`, `internal/driftengine/doc.go:548-552` and `internal/driftengine/verify.go:1-4`; those files were not in my context. Line 121 of `prreview.go` came from a grep in the same call.
