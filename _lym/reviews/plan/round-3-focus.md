---
round: 3
focus:
  - "Cards that make `internal/landingshed` production code import a package its import allowlist (`landingshedAllowedImports`, checked by `TestToldGeometryInvariant_AllowlistOnly` in `internal/landingshed/seam_enforcement_test.go`) does not list, such as `internal/pullrequest`, `internal/githubclient`, `internal/parentdirective` or `internal/editdirective`: settle whether a card lists the allowlist and its test under Edit and states the added imports."
  - "The import guard card 16 adds, `TestPullRequestGuard_OnlyLandingAndTeardown`, walks every non-test `.go` file with `scankit.Walk`: settle whether every package the plan makes import `internal/pullrequest`, including `internal/testkit/envkit`, is on the guard's allowed list."
  - "The `PR-Review` row's `Call` removal of a record marked `ClosedForContinue` before `wait`, and `wait`'s closed-and-`ClosedForContinue` Done verdict: settle whether the verdict is reachable by a named test row, and whether a re-entry after a crash between recording `ClosedForContinue` and returning Done keeps the operator's ending."
  - "The mail the `PR-Review` wait posts to the agent when the pr-review gate fails: settle whether the reason text it carries names the failing tier and the log path as the decision record requires, against what `verifyGate.check` in `internal/landingshed` actually returns, and whether the wording suits an agent in a live session."
  - "Docs the plan's cards falsify: `README.md`'s Landing bullet on `PR-Review` and the `internal/loomrecipe/doc.go` sentence on `PR-Review` ending Done with no `on_stuck`: settle whether each is on a card's target list with its rewrite stated."
  - "The wait's poll cadence: settle whether every extra poll the plan adds beyond the decision record's at-most-once-a-minute GitHub poll bound is recorded in the overview's Shared Decisions and pinned by a named test row."
---

Focus entries name where to look; the reviewer still judges the whole plan against the rubric.
