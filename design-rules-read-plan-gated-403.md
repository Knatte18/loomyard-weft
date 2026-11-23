# Landing blames the token when the branch-rules read returns a plan-limit 403

`rules-read-plan-gated-403` · Note · bug, landing, git

Imported from [issue #179](https://github.com/Knatte18/loomyard/issues/179).

## What happened

`lym loom` halted `blocked` at the Finalize step with anomaly `escalation-to-human`.
The landing step reads the parent branch's rules through `GET /repos/<owner>/<repo>/rules/branches/main?per_page=100`.
GitHub answered 403 with the message "Upgrade to GitHub Pro or make this repository public to enable this feature."
lym's landing stop reported it as a refused token:

```
landing stop: GitHub refused the token for the call to read GitHub's rules for the parent branch: pullrequest: read the rules active on branch main: GET https://api.github.com/repos/<owner>/<repo>/rules/branches/main?per_page=100: 403 Upgrade to GitHub Pro or make this repository public to enable this feature. []; way forward: make sure the gh user's token (GH_TOKEN, GITHUB_TOKEN, or gh auth login --hostname github.com) is valid and can reach the repository, then run lym loom resume
```

Every earlier step (Preflight through PR-Review) was done or skipped cleanly; the run stopped only at landing.

## Why it is abnormal

The 403 is a plan limitation, not a token problem.
The token is valid and reached the repository; the branch-rules feature is unavailable for a private repository on a free plan.
The way forward lym prints (check the token, then `lym loom resume`) cannot succeed, so the run stays blocked on a loop the operator cannot escape from that advice.
A repository with no rules feature has no rules to enforce, so landing should not need this read to succeed.

## What to do

- Distinguish the 403 "Upgrade to GitHub Pro or make this repository public" response from a token refusal in the pullrequest rules read.
- Treat that response as "no active rules" and continue landing, possibly with a note, or
- stop with a way forward that names the plan limitation (use a public repository, upgrade the plan, or opt out of the rules read) instead of the token advice.

Provenance: task `greet-goodbye`, from note `loom-halt.md`.
