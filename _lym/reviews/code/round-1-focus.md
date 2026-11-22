---
round: 1
focus:
  - "internal/battencli/arm.go `hubRunVerdict`/`judgeAfterCodeTipFetch`: does `fetchRunForCatchUp` fetch the records repository (the `-weft` remote-tracking ref), so the prune reaches the stale `origin/<slug>-weft` ref? Settle that the second judge reads fresh refs rather than a cached read."
  - "internal/gitrepo/pull.go `Fetch` now runs `--prune`: which callers of `Fetch` read remote-tracking refs that a prune can now drop (reach lists many tests, including fabriccli and fabricengine)? Settle whether any caller relied on a deleted-branch ref staying."
  - "internal/runfsm/act.go `codeTipRefusal` (mutation survivor at act.go:202, a dropped statement): does `TestAct` assert both the reason and the way-forward text for the pair-absent case, and for the machine-named and machine-absent variants?"
  - "internal/pairteardown/runread.go `CodeTipPairAbsent = !pairHere`: is `pairHere` the right condition, and does it stay false when a pair stands? Settle whether `TestReadRun` pins both values."
  - "Scope against the board entry's Expected list: the diff prunes on fetch, reads fresh in the act check and rewords the refusal; it does not change `lym batten cancel --remove`. Settle whether the first bullet's either-or and the refusal wording bullet are both met."
  - "Commit trailers: check each `Root-Cause:` trailer names a defect the diff removes, and that `TestJudgeAfterCodeTipFetch` covers the failed-fetch path, the read-error path and the non-code-tip path without recomputing expectations."
---

Seed focus from the committed diff and the board entry under `## Goal`; the decision record's other sections are empty.
