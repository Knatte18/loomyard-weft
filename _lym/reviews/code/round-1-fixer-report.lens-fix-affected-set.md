```yaml
lens: fix-affected-set
outcome: reported
```

Task: the fix-affected-set lens. I re-ran the search behind each finding whose premise is a set: F1's literal sites, F2's callers, F10's clone-command sites, and F3's reads.

Findings:

- LOW. `internal/fabriccli/clone.go:34`, a doc comment the fix did not touch.
  - It still says "...into origin under refs/lym/".
  - F1's fix rewords `doc.go` "so the package spells no namespace", but this comment is another literal in `fabriccli`.
  - It sits in the finding's set by key but is outside the review's named locations.
  - The search `grep -rn "refs/lym" internal/fabriccli` (non-test) returns only this line after the fix.
  - The fixer may reword it to "under origin's records namespace".

Checked and covered, no finding:

- **F1:** `fabric.go` and `doc.go` no longer spell `refs/lym/`. The only remaining non-test literal is the `clone.go:34` comment above.
- **F2:**
  - There are four `ErrOriginUnreachable` wrap sites. `catchup.go:169` and `catchup.go:188` wrap Bolt pull and registration errors, which are not layout reads, so they are correctly untouched.
  - `catchup.go:231` is the fixed site.
  - `arm.go:642` wraps a plain code `Fetch()`, not `FetchPairTips`, so it is not affected.
  - The other `FetchPairTips` and `ReturnToPreviousTransition` callers are `battencli/codetipsettle.go:53` and `:75`. Both return the error unchanged, so the layout error with its refspecs reaches the operator.
  - `arm.go:638` (`hubRunVerdict`) goes through `fetchRunForCatchUp`, which now passes the layout error through.
- **F10 and F5/F6:** `cloneResetCommand` has the three `existingHubError` callers (`clone.go:305`, `:344`, `:397`) plus the three uses in `RefuseHubWithoutBolt`. All pass the full `opts`, so `--force-bootstrap` is carried everywhere. The seed and join forms deliberately set no `ForceBootstrap`.
- **F3:** the review named four layout reads. `FetchPairTips` and `ReturnToPreviousTransition` are now covered by `TestOneRepositoryPairTips_RefuseARecordsStoreThatLostItsFetchRefspecs`, and `resolveMergeSources` was already covered. The `fastForwardWeft` read has no test, which the fixer must report as fixed-differently.

Unchecked: F7's cancel-passes-its-own-command row (no test reaches `hubCancelDeps`' `DiscardTaskBranch`), F8 and F9 (summary text), and F4's envelope key beyond the `plain_archive_tags` assertion. Those are not set premises, or fell outside my call cap.

Tool calls:
1. One bash call, four greps: `refs/lym` in `fabriccli`, callers of `FetchPairTips` and `ReturnToPreviousTransition`, `ErrOriginUnreachable` wrap sites, `cloneResetCommand` and `existingHubError` sites. Searches over the whole tree were not in my context.
2. One bash call, `sed` of `battencli/codetipsettle.go:45-85` and `arm.go:625-650`, plus a `clone.go` grep. I needed to see whether the other callers of the changed functions wrap their errors.
