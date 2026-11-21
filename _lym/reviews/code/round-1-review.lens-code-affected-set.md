```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, I traced every caller, test, doc and rule that reads or describes it.

**Affected set per change**

- **`rulesFeatureUnavailable` and the early return in `RequiresPullRequest`.** A plan-gated 403 now returns `false, nil`; any other error still returns a wrapped error.
  - `landingshed.readParentRule` → `resolveWithParentRule` (`internal/landingshed/pullrequestmode.go:193-219`): covered. With no error it falls to `!required` → `decision`, and the halt no longer fires. Its doc comment (lines 176-183) speaks of "failure of the read", and a plan-gated 403 is no longer one, so it stays true.
  - `beamcli.parentRuleRequired` (`internal/beamcli/pr.go:480-497`): covered. The call now returns `false, nil` with no warning, and its doc comment (lines 474-479) stays true.
  - `internal/beamcli/doc.go:209-211` and `internal/beamcli/land.go:54-55`: covered. Both say a failed or tokenless read reads as no rule, and that stays accurate.
  - `landingshed.githubHaltReason` and its doc comment (`internal/landingshed/finalize_pr.go:67-83`): covered. "a 401 or 403 response names the token" still holds for every 403 that now reaches it, and the plan-gated one no longer does.
  - `internal/landingshed/doc.go:149-157`: updated in the diff and consistent with the code.
  - `internal/pullrequest/doc.go:23-28`: updated in the diff and consistent.
  - Test fakes (`prkit.API`, landingshed `fakePullRequestAPI`, pairteardown `closeRecordingAPI`): the interface signature is unchanged, so no change is needed. Covered.
  - `docs/overview.md:1376` describes the resolver at a level the change leaves true. Covered.
  - `docs/research/single-repo/sandbox-2-tasks-2026-11-21/report.md:111,120` records a past run, so the measurement-report exception applies. Covered.
- **The early return in `RequiredChecks`.** A plan-gated 403 now returns `nil, nil`.
  - `pullrequest.ReadMergeState` (`internal/pullrequest/mergestate.go:52,78-80`): covered. No checks now means no checks block the merge, with no error. The doc comment at line 52 ("reads … the required checks of base") stays true.
  - `internal/pullrequest/doc.go:37-41` (`RequiredChecks`): it still describes the read without the 403 case. Line 28 of the same doc covers the case for "Both rules reads", so this is covered.
- **The new constant `planGatedMessagePrefix`.** `PATTERN-code-conventions` asks for a name that can go stale to be one named constant. It is declared once, and the test fixture's full GitHub message is a server-reply literal, not a use of the name. Covered.
- **The sandbox-3 report anchor.** The target heading is `## 3. The first hub after the pull` (report line 100), and the new anchor `#3-the-first-hub-after-the-pull` matches it. Nothing else reads the anchor. Covered.
- **Declared PATTERN changes:** the directive declares none, so there is nothing to land.
- **Commit messages:** the one `Root-Cause:` trailer, on `7e727a15`, names the defect the diff removes: the rules reads returned the plan-gated 403 as an error. That is not a symptom. No `Pattern-Departure:` or `Loosened-Check:` trailer appears, and none is needed. No message claims agreement with a reference or an improvement.
- **Stencils:** the diff changes none.

**Findings**

None. Every member of the affected set is either updated or still accurate.

**Tool calls**
1. Grepped the repo's `.md`, `.go`, `.tmpl` and `.yaml` files for docs and code that describe the rules read. My context had only call sites, not the prose that describes this behaviour.
2. Grepped those files for lines naming the rules read, 403 or required checks, to locate the descriptive lines.
3. Read `internal/beamcli/doc.go:205-222`, `docs/overview.md:1370-1395` and `internal/beamcli/land.go:48-60`. These docs describe the rules read's outcome and were not in my context.
