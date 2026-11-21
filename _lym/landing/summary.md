# Read GitHub's plan-gated 403 on a rules read as no rules

A private repository under a free GitHub plan answers `GET repos/{owner}/{repo}/rules/branches/{branch}` with a 403 opening "Upgrade to GitHub Pro or make this repository public to enable this feature."
GitHub answers it for every token, so it says nothing about the token.
`landingshed.githubHaltReason` mapped every 403 to a token failure.
A run that already held a pull request record therefore halted at Finalize, naming the token,
and the way forward it named, a valid token and `lym loom resume`, could never work.
A run with no pull request record logged the same 403 as a warning and landed.

`internal/pullrequest` now reads this 403 as "no rules apply", since a repository without the rules feature has no rules:

- `RequiresPullRequest` returns false.
- `RequiredChecks` returns no checks, without reading check runs or commit statuses.
- The `internal/landingshed` and `internal/pullrequest` package docs state the behaviour.

Also fixes the sandbox-3 report's link to its numbered heading anchor.

## Design decisions

- The 403 is recognised by status and message together: `rulesFeatureUnavailable` requires a 403 whose message starts with `Upgrade to GitHub`.
  Any other 403, such as "Resource not accessible by integration", still fails the read and still halts a run that holds a pull request record.
- The fix sits in `internal/pullrequest`, at the one place both rules reads share the call, so `landingshed` needed no change beyond its doc.
- The message prefix is GitHub's wording, not a documented code.
  If GitHub rewords it, the old halt returns for free-plan private repositories, and nothing else changes.

## To check by hand

- Not run against GitHub: rerun a task on the free-plan private sandbox repository (`Knatte18/lym-sandbox-onerepo`) with `pull_request` off and a pull request record held, and confirm Finalize lands.
  The tests cover the 403 against a scripted server.

## Merge danger

**Door:** two-way
**Blast radius:** pullrequest
