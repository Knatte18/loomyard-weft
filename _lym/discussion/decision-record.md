## Goal

```markdown
Imported from issues #176 and #177; read both with `gh issue view <n> --repo Knatte18/loomyard` for the full reports.

## Fixer reruns a finished round (#176)

Plan-Burler-Fix started a second fixer session for round 1 after the first had fixed every finding.
The first fixer's report sat under a timestamped name, not `round-1-fixer-report.md`, so the round looked unfinished, and the prompt gave no reason for the rerun.
Fix: accept the finished report, or state in the fixer prompt why the step runs again (misnamed or rejected report versus new work).

## Fork git in its own scratch repo (#177)

A fixer fork ran `git init`, commit and push in a throwaway repo under `.scratch/` to probe push behaviour; the mutating-git guard ended the run as a hard failure, and its way forward said to revert, with nothing to revert.
Fix: scope the guard to the target repository and its remotes, so git in a repo the fork created under `.scratch/` passes; make the halt say when no target change was found.

## Triage against main (2026-11-21)

- Rerun: `internal/shedadapters/burlerfix.go` renames a leftover round fixer report to a timestamped name before each attempt; the fixer prompt never says why the step runs again.
- Fork git: 27498365 lets a fork run git in a repository it created, but `internal/burlerengine/cluster.go` still refuses every push, and the halt text still says revert.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - clone-of-remote-git-named-repo-sets-redirect-flag (class design, severity MEDIUM, open in rounds 1)
  - config-env-attached-form-bypasses-redirect-check (class design, severity BLOCKING, open in rounds 1)
  - failure-exit-without-report-records-no-reason (class design, severity MEDIUM, open in rounds 1)
  - halt-claims-target-repository-unchanged (class design, severity MEDIUM, open in rounds 1)
  - lexical-ownership-admits-symlinks-and-failed-clones (class design, severity MEDIUM, open in rounds 1)
  - pre-attempt-archive-record-untested (class design, severity MEDIUM, open in rounds 1)
  - relative-push-path-resolved-from-wrong-directory (class design, severity BLOCKING, open in rounds 1)
  - remote-tracking-marks-safe-while-git-pushes-elsewhere (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
