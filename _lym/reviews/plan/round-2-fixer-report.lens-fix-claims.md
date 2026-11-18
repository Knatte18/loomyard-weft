```yaml
lens: fix-claims
outcome: reported
```

Task: check the claims the fixer's changes make against the code at HEAD.

No findings.

- **Card 3:** the claim that `NilSeams` skips the paths `nilLegal` lists holds; `walkNil` checks `nilLegal[path]` on the full dotted path (`internal/testkit/envkit/envkit.go:799`). At HEAD, `nilLegal` holds only the top-level `ReviewBase`, not `Drift.ReviewBase`. The new sentence is still right, because card 3 already edits `envkit#nilLegal` to list `Drift.ReviewBase`.
- **Card 3:** the reason "loom- and darn-only seam, batten has no Drift row" matches every existing `Drift.*` entry (`internal/battencli/wire_seams_test.go:180-199`).
- **Card 9:** the reason "loom-only seam, batten lands no branch" and the claim that batten leaves `Landing.PullRequest` unwired both match the table. Every `Landing.PullRequest.*` seam is listed nil, and only the beam-only entries have a different reason (`wire_seams_test.go:113-133`).
- **Card 7:** `BurlerHalfRunner.GotReviewProfiles` exists (`internal/testkit/shedfake/shedfake.go:215`). `newTestEnv`'s `env.Burler` is that fake, since `captureReview` already casts it (`entries_burlerhalves_test.go:83`). `burlerReviewEntry(row, cfg, env)` builds the producer directly (`entries_burler.go:73`). So the reworded row can be carried out without `captureReview`.

Tool calls:
- Read `envkit.go:55-75` and `:778-805`. My context showed only grep hits naming `nilLegal`, not its keys or how `NilSeams` matches paths.
