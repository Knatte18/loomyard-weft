```yaml
lens: code-affected-set
outcome: reported
```

**Task:** list the set each change forces outside its own lines, and flag anything the diff leaves wrong. I found no findings.

**Per change: the set found and whether it is covered**

1. **`pullrequest.API` gains `Reopen` and `RequiresPullRequest`.** Covered.
   - Implementers are `Client` (`internal/pullrequest/client.go`), `fakePullRequestAPI` (`prreview_test.go:92`) and `closeRecordingAPI` (`teardown_test.go:1202`).
   - Both fakes gain the two methods.
   - The `envkit.go:214` and `landingdeps.go:321` seams are func literals returning the interface, so they need no change.
   - The `cmd/lym/ghguard_test.go:98` mutation list names `.PullRequests.Edit(`, which `Reopen` uses, and the check is package-scoped, so it still passes.
2. **`ResolvePullRequest` gains `parentRuleRequired`.** Covered. No production caller of `ResolvePullRequest` exists outside `pullrequestmode.go`, apart from `resolveWithParentRule`, which is the only caller. No test outside `pullrequestmode_test.go` calls it.
3. **`Record.ClosedForContinue` becomes `ReviewEnded`, JSON key unchanged.** Covered. A repo-wide grep finds no remaining `ClosedForContinue` or "closed for continue", and the `prcontinue.go` references and tests are updated.
4. **Finalize's merge-refusal halt text and the closed-PR halt text change.** Covered. Finalize and PR-Review test rows assert the new texts, and no stale reference to the removed wording remains.
5. **`continueComment` and PR-Review's `api.Close` call on the continue flag are removed.** Covered. A grep of non-test code finds no leftover reference to `continueComment`. The `judge` signature drops `api` and `state`, and its only caller is updated.
6. **Docs, help and stencil.** Covered.
   - Updated: `landingshed/doc.go`, `finalize.go`, `finalize_pr.go`, `pullrequest/doc.go`, `record.go`, `prcontinue.go` (header, Long help and note), the PR-Review stencil and `docs/overview.md`.
   - Left as written: `loomcli/doc.go:255` says only that the verb sets the flag, and the `shellguard/rows.go:148` deny text says "opens, merges or closes", which stays accurate.
   - The sampler stencil has no match.
   - The remaining "lands locally" mentions in `doc.go`, `finalize.go` and `finalize_pr.go` describe the no-pull-request path, which is correct.
7. **PATTERN.md.** Covered. `PATTERN-github-auth` adds "reopen" and `PATTERN-pull-request-mode` names the parent's GitHub rules and the merge on GitHub. Both have no `pattern/` background file. The rubric has no declared-changes list to check, so there is nothing to raise.
8. **Commit message.** Covered. It carries no `Pattern-Departure:` trailer and the diff needs none.

**Tool calls (1):** one combined grep, run to find other `API` implementers, other `ResolvePullRequest` callers, leftover `ClosedForContinue` and stale "closes"/"lands locally" wording, and the `ghguard_test.go` call list. The manifest did not hold a repo-wide search for these.
