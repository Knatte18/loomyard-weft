```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every claim the fix added or rewrote against the code at this worktree's HEAD.

no findings

Claims confirmed against the code:
- **D2:** the fork precondition holds. `beamForkPrecondition` in `internal/battencli/beamfork.go` gates the creation of a member's pair, and D3's refusal 2 means no member pair exists when the beam is flagged ready.
- **D8, merged at another head:** "pulling the parent first so its merge-in carries GitHub's merge" holds. Finalize decides about the pull request at step 1b2b and only then runs `syncParent` at step 1b4, which fast-forwards the parent to its upstream before the merge-in (`internal/landingshed/finalize.go`).
- **D8, the scratch record:** "Finalize keeps a pull request it finds or opens in its scratch pull-request record" holds; `openAtOnce` records it (`internal/landingshed/finalize_pr.go`).
- **D8, the retry window:** the one-minute window holds (`prMergeRetryWindow`).
- **D8, the mismatch:** "the mismatch PR-Review halts on today" holds (`internal/landingshed/prreview.go`).
- **D11 bound:** after the repair cap, an interrupted stop under the re-invoke policy falls to the tackler row. The arbiter follows when the tackler's resume returns to the same stop (`decideDrive` in `internal/sheddriver/driverules.go`).
- **D11, the arbiter:** the claim that its existing rule admits the stop holds. `arbiterstop.Eligibility` lets through an interrupted stop under the re-invoke policy, whatever the recipe.
- **D13 and the Constraints line:** the driver's rule table hands a paused run back to the parent. That row comes before every tackler and arbiter row.
- **D13, the reopen commit:** "as `lym beam reopen` commits the landing directory today" holds (`internal/beamcli/doc.go`).
- **Constraints, the dobby:** `State.Land` is cleared only on a reading that does not pass. An episode recorded after a successful `Land()` is not marked refused, so no dobby, restarted or not, writes the land request again.
- **Constraints, refusal classes:** `classifyRefusal` returns RefusalPending only for "base branch was modified", "mergeability" and "still being computed". A pending required check and a dismissed review fall to RefusalOther.

Tool calls beyond my context:
1. Grep and read of `driverules.go` `decideDrive`, `JudgesFirst`, the beam fork precondition and the record lines in `finalize_pr.go`: my context lacked the rule table past the cap, the arbiter seam, the fork's gate and how Finalize records a pull request.
2. Grep of `finalize_pr.go` for its record handling: the first grep's pattern missed Finalize's recording.
3. Grep of `finalize.go` for fetch and the pull-request decision: needed to know when Finalize syncs the parent.
4. Read of `finalize.go` around its step order and `syncParent`: needed to confirm the decision comes before the parent fast-forward.
