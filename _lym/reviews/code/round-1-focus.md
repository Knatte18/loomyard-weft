---
round: 1
focus:
  - "internal/hubreconcile/walk.go reconcileHubWide: does `written` carry the overview into the commit's path list the way the config files go, so a changed HEDDLE.md is committed and an identical render commits nothing, including when the config reconcile itself wrote no other file?"
  - "walk.go reconcileHubWide: the overview is rendered after the hub-wide reconcile but before afterHubWideWrite and the commit; settle that every failure path after the render (commit failure, pull skip, hook error) restores HEDDLE.md through the snapshot, and that a skipped walk (board cannot be brought up to date) renders nothing."
  - "walk.go renderHeddleOverview: a failed render is logged and swallowed; settle whether RenderOverview can leave a partial or changed file on error, so 'leaves the file as it was' holds, and that the Warn call follows the repository's logging conventions (way_forward key, message form)."
  - "Does the walk render under the Bolt write lock that heddle set/remove and the Bolt replayer take, with no lock-ordering or re-entrancy problem against RenderOverview's own locking?"
  - "Scope against the decision record's Goal: set/remove rendering is unchanged, and the hook chosen runs after the landing and after a lym update; check the diff makes no change beyond the walk, its docs, the import allowlist and PATTERN-heddle-instructions."
  - "internal/hubreconcile/ensure_integration_test.go new test: does it duplicate coverage of an existing restore-on-failure or walk-commit test, and does each case assert behavior the diff added? Check the edit to heddlecli cli_integration_test.go still pins that set/remove leaves the overview for the board sync."
  - "Repository rules: seam_enforcement_test allowlist change, PATTERN.md / pattern/ doc updates, and package docs landing in the same diff; check summary.md claims (stamp-gated once per build, restore on failed board write, Warn on render failure) against walk.go."
---

Seed focus for round 1 of the code review of the HEDDLE.md render in the hub config reconcile.
