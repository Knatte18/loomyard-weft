# Branch-rules 403 on a free private repo reads as no rule

`rules-403-plan-reads-no-rule` · Note · bug, landing

## Seen

In the one-repository sandbox hub (private repository on a free GitHub plan), `pull_request` was turned off at the default layer while the run greet-goodbye already held an open pull request.
Finalize then read the parent branch's rules, `GET repos/{owner}/{repo}/rules/branches/main`, and GitHub answered 403 "Upgrade to GitHub Pro or make this repository public to enable this feature".
Finalize took that as an unusable token and halted, since the run held a pull request record.

## Idea

That 403 is a definite answer, not a token fault: repository rules are not available on that plan, so none are active.
Read it as no rule in `RequiresPullRequest` and `RequiredChecks` (`internal/pullrequest/client.go`), and keep the halt for other 403s.
