# Finalize halts on a private free-plan repo: rules read 403 read as a bad token

`rules-read-plan-gated-403` · Note · bug

Found by onerepo-sandbox-tasks (sandbox `Knatte18/lym-sandbox-onerepo`, private, free plan).

The sandbox task greet-goodbye halted at Finalize after PR-Review:

```
landing stop: GitHub refused the token for the call to read GitHub's rules for the parent branch: pullrequest: read the rules active on branch main: GET https://api.github.com/repos/Knatte18/lym-sandbox-onerepo/rules/branches/main?per_page=100: 403 Upgrade to GitHub Pro or make this repository public to enable this feature. []; way forward: make sure the gh user's token ... is valid and can reach the repository, then run lym loom resume
```

The token is fine: GitHub gates repository rules behind a paid plan for private repositories, so the endpoint answers 403 for every token.
`landingshed.githubHaltReason` maps every 403 to a token failure, and `pullrequest.RequiresPullRequest` and `RequiredChecks` surface it as an error, so a run that holds a pull request record halts.
The way forward it names cannot work: a resume hits the same 403.

Fix: read this plan-gated 403 as "no rules apply" (a repository with no rulesets feature has no rules), or at least name the real cause and a way forward that works (make the repository public, or land with `pull_request` off on a repository with no PR record).
Every private repository on a free account hits this at Finalize.