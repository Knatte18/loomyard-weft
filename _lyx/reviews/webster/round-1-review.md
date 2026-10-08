---
verdict: APPROVED
---

# Webster-Review round 1

Range: `product.parent` is `main`, and `_lyx/loom/rework/round-1/record.json` carries `class: required` and `first_card: 10`, which equals the plan's `first_card`.
The live round's `head_sha` is `2ecad53`, so the subject is `git log --first-parent --no-merges -p 2ecad53..HEAD`: one commit, `5e2afb7b8`, against card 10.

No findings.
The focus questions settle as follows.

- **`template.yaml`.** `review` is `opus[medium]` and `webster_review` is `sonnet[high]`.
  `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix` and `webster_fix` are still `""`, `fix` is still `opus[medium]` and `judge` is still `sonnet[medium]`.
  The comment on each changed line is untouched and still correct: `review` names the run-wide reviewer list, and `webster_review` states that an empty value takes the run-wide list and a set value replaces it for the segment.
- **`wiring_test.go` `TestWire_ReviewKeysReachTheEnv`.** The loop over the run-wide rows is split so it covers `NameDiscussionBurler` alone.
  A new assertion pins the Webster-Burler row's `Review` as one `sonnet` entry of effort `high`, and a second pins its `Fix` equal to `c.env.ReviewModels.Fix`.
  The doc comment states both.
  The Plan-Burler assertions are unchanged.
- **`config_test.go` `templateConfig`.** `Review` is `ModelSpecList{"opus[medium]"}` and `WebsterReview` is `ModelSpecList{"sonnet[high]"}`.
  The other five per-segment lists remain `ModelSpecList{""}`.
- **Other pins of the old template values.** A grep over the tree finds no other test, fixture or doc that pins the template's `review: sonnet[medium]` or an empty `webster_review`.
  The remaining `sonnet[medium]` and `opus[effort=high]` hits are seeded legacy files and `Config` literals that set their own values, not the template.
- **Test economy.** The wiring assertion edits an existing test rather than adding one.
  It covers a path no other test reaches: `internal/loomengine/review_test.go` sets `WebsterReview` to `ModelSpecList{""}`, so only this test drives the template's `webster_review` through `resolveSegmentModels` into `RowReviewModels`.
  `templateConfig` covers the template-to-`Config` step and the wiring test covers the `Config`-to-env step, so they do not overlap.

`go test ./internal/loomengine/ ./internal/loomcli/ ./internal/configreg/` passes, and `go vet` on the first two is clean.
