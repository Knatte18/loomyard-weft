---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: scope
    origin: lens:code-scope
    location: _lym/landing/summary.md#Change
    summary: The change description omits the sandbox-3 report anchor fix, a gate-forced edit outside the Goal.
    fix: "Keep the anchor edit in docs/research/single-repo/sandbox-3-reclone-2026-11-21/report.md. Add one bullet under `## Change` in _lym/landing/summary.md saying the sandbox-3 report's link now points at the numbered heading anchor `#3-the-first-hub-after-the-pull`, because the docs-link gate rejected the merge base's broken anchor."
---

### [NIT] Change description omits the gate-forced anchor fix

**Location:** _lym/landing/summary.md#Change
**Issue:** Commit `03603d1c` changes a link fragment in the sandbox-3 report that no Goal point asks for.
The edit is forced:
the merge base links to `#the-first-hub-after-the-pull`, the heading at `report.md:100` is `## 3. The first hub after the pull`,
and the docs-link guard in `internal/lymcwd/docslink_test.go` fails on a missing anchor.
The edit stays, but the change description, which becomes the landing text, never mentions it.

## Assessment

The diff makes the change the Goal asks for first: a plan-gated 403 on `GET rules/branches/{branch}` reads as "no rules apply".

- `rulesFeatureUnavailable` (`internal/pullrequest/client.go:28`) matches only a `*github.ErrorResponse` with status 403 and a message opening `Upgrade to GitHub`.
  go-github's `CheckResponse` returns a rate-limit 403 as `*RateLimitError` or `*AbuseRateLimitError`, not as `*ErrorResponse`, so neither matches.
  githubclient's 401 path returns `ErrTokenRejected` with no response, which doesn't match either.
  `errors.As` reaches the response through the `url.Error` and `%w` wrapping, and the nil-response guard holds.
  A real token failure therefore still errors, and the `forbidden` rows pin that.
- Later pages: an early return on page 2 or later would drop what earlier pages gathered.
  This can't happen, because a page that succeeds means the repository has the rules feature.
  Not a defect.
- Callers: `resolveWithParentRule` now gets `false, nil` and returns the no decision, so the sandbox's halt is gone.
  `beamcli.parentRuleRequired` loses only its warning, and `ReadMergeState` sees no required checks.
  `githubHaltReason` still maps every other 403 to the token, and its doc comment stays true.
- Covering `RequiredChecks` is within scope: the Goal's reason, that a repository without the feature has no rules, applies to every read of that endpoint.
- Tests: the `plan` and `forbidden` rows check what their names say.
  The `unknown` assertion for `RequiredChecks` moved into a loop but was not loosened.
  The mutation report has no survivors, and the redundancy report lists `TestClient_Calls` as unique.
- Docs: `internal/pullrequest/doc.go` and `internal/landingshed/doc.go` state the new reading.
  `PATTERN-code-conventions` is met because the prefix is one named constant.
  No PATTERN change is declared.
- Commits: the `Root-Cause:` trailer on `7e727a15` names the defect the diff removes, not a symptom.
  No message claims agreement or improvement.
- Merge danger: `two-way` / `landing` holds.
  The diff changes no record format or config meaning and adds no push of its own.
  It routes a plan-gated run into Finalize's existing close-and-land path.

## Rejected

- Assistant F1 (MEDIUM, missing tests for a non-403 upgrade message and a later-page plan 403): GitHub sends neither answer. A later-page plan gate is unreachable, and the test rules exclude states the dependency's guarantees rule out. The mutation report shows no surviving mutant on the changed lines.
- Assistant F2 (LOW, revert the sandbox-3 anchor fix): the docs-link guard in `internal/lymcwd/docslink_test.go` rejects the base's broken anchor, so the edit is forced, and reverting it would fail the gate. The omission it points at is kept as F1.
