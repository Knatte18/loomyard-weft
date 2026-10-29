```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every way forward, halt and next step the fix added to the decision record against the code.

1. **[MEDIUM] The ungated-merge halt's way forward leaves the board entry reading as unlanded, though the work is on the parent.**
   - Changed text: D9, the bullet "merged, with a merged head other than the record's gated head", whose way forward ends in `lym batten cancel <slug>`.
   - Problem: the pull request is already merged into the parent. Cancelling tears the pair down unlanded, so the board clears the entry instead of marking it done. The entry then reads as unrun work that was in fact landed, and a rerun would start over work the parent already holds.
   - Evidence: `internal/pairteardown/doc.go` "Board claim". Whether the work landed is read from "the request's Landed flag, or a Finalize with outcome Done in that status file's history". Otherwise "an entry whose ... batten run is paused, blocked, failed or awaiting, is cleared". A halted run has no Finalize Done, so cancelling clears the entry.
   - Already present: the "merged, with task commits the merged head lacks" bullet has the same cancel way forward. The fix copied that pattern into the new halt.

2. **[LOW] The dirty-tree merged halt offers "committing" as a way forward, but that path only reaches a second halt.**
   - Changed text: D9, the bullet "merged, with the merged head equal to the task head, on a dirty tree", way forward "committing or discarding them in the task worktree, then `lym loom resume`".
   - Problem: after a commit and resume, the task has a commit the merged head lacks. That triggers the "task commits the merged head lacks" halt, which ends in `lym batten cancel <slug>`. Only discarding reaches Done. The halt text does not say that committing hands the files off to a board note and a cancel rather than to a landing.
   - Evidence: D9's own bullets, read in order after the resume.

Checked and found sound:
- D4's two split ways forward.
- D11's per-refusal ways forward. `squash` is `CategoryCrossPair` (`internal/landingshed/config.go:115`), and `lym config <module> --set` writes the hub file (`internal/configcli/doc.go:4`), so the `squash` way forward can be carried out.
- The no-closes-the-pull-request steps in D3 and D11.
- D10's status read, which can be reached while the run status reads running at `PR-Review`.

Out of scope for this lens: D7 says the identifiers are written "when it starts the session". A crash between the start and the record write leaves a live session with no record, so a re-entry starts a second one. The fix-claims lens owns this.

Tool calls:
- `grep` of `internal/landingshed/config.go` and `internal/configcli/doc.go`: my context did not show the `squash` key's category or how `lym config --set` writes.
- `sed -n 55,70p internal/pairteardown/doc.go`: my context did not show how a teardown settles the board entry after a cancel.
