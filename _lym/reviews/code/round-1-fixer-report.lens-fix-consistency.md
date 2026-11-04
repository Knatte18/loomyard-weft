```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens — whether the change set still agrees with its twins and surroundings.

**Findings**

1. **MEDIUM** — `internal/landingshed/doc.go:258`, the package doc for the closed-record halt.
   - The text still reads: "a yes whose pull request GitHub reports closed while the record carries no operator's ending is Stuck with Halt naming a reopen on GitHub and `lym batten cancel`".
   - The fix changed `finalize_pr.go:206` to `!rec.ReviewEnded && !decision.MergeAtOnce`, so a closed recorded pull request under a merge parent no longer halts. It is reopened and merged.
   - Line 258 lists no MergeAtOnce exception for this case. Line 259 also says only a record with the operator's ending is reopened.
   - The fixer updated the `Call` doc in `finalize.go` and the `pullRequestLanding` comment, but not this twin.

2. **LOW** — `internal/landingshed/doc.go:136`, the protected-branch way-forward paragraph.
   - It says the refused-push reason always carries the `pull_request=merge` setting advice.
   - After the fix, `protectedParentWayForward` leaves that advice out when `parentMerges` is true, so the doc no longer matches.
   - `finalize.go:176` and the `pushParent` comment were updated; this one was not.

3. **LOW** — `internal/landingshed/doc.go:263`, the MergeAtOnce squash line.
   - It says "A yes marked MergeAtOnce always merges by squash, whatever `squash` says."
   - The parent-side merge now squashes too, and that behavior (F1) is not stated in the doc.
   - The line still reads correctly for the GitHub merge. It is stale only by omission, so it is a nit-level gap.

**Checked, consistent:** the `Config.Squash` uses at `finalize.go:337` and `finalize_pr.go:273` agree. Both test callers of `protectedParentWayForward` were updated with the third argument. The remaining halt text at `finalize_pr.go:207` still applies to non-merge modes, which is correct.

**Tool calls**
- `grep` for "operator's ending" and "closed without a merge" across `.go`, `.md` and `.yaml`: the context lacked any sibling text outside the changed files.
- `grep` for `Config.Squash` in non-test code: to find twin squash sites.
- Read `doc.go:255-264`: the context lacked the package doc.
- `grep` for "protected branch" in `internal` and `contracts`: to find twin way-forward texts.
- Read `doc.go:136`: to confirm the paragraph wording.
