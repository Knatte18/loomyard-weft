# Finalize halts on a private free-plan repo: rules read 403 read as a bad token

`rules-read-plan-gated-403` · Task · bug, landing

Found by onerepo-sandbox-tasks (sandbox `Knatte18/lym-sandbox-onerepo`, private, free plan); report at `docs/research/single-repo/sandbox-2-tasks-2026-11-21/report.md`.

The sandbox task greet-goodbye halted at Finalize after PR-Review had opened PR #1:

```
landing stop: GitHub refused the token for the call to read GitHub's rules for the parent branch: pullrequest: read the rules active on branch main: GET https://api.github.com/repos/Knatte18/lym-sandbox-onerepo/rules/branches/main?per_page=100: 403 Upgrade to GitHub Pro or make this repository public to enable this feature. []; way forward: make sure the gh user's token ... is valid and can reach the repository, then run lym loom resume
```

Trigger: the orch set `pull_request` off at the sandbox's default layer while the run already held PR #1, so Finalize resolved "no pull request" and read the parent branch's rules.
A run that holds a pull request record halts on any failure of that read.

The token is fine: GitHub gates repository rules behind a paid plan for private repositories, so the endpoint answers 403 for every token.
`landingshed.githubHaltReason` maps every 403 to a token failure, so the halt names the token,
and the way forward it names cannot work: a resume hits the same 403.
A run with no pull request record logs the same 403 as a Warn and lands, as the sandbox's later tasks did.

Fix: read this plan-gated 403 as "no rules apply" (a repository without the rules feature has no rules),
or at least name the real cause and a way forward that works, such as setting `pull_request` back to on so the run merges its pull request.
