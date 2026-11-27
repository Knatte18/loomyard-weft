```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking whether each set-shaped finding's fix covers every member.

**Findings left unchecked:** none of the set-shaped ones (F1, F2, F3, F7, F12, F13).

**Findings**

1. **LOW**, F13 fix (`beamsetting.go`, `dobbyengine/config.go`): two sites with the same "operator's own ending" wording were left unchanged.
   - `internal/dobbyengine/reinterpret.go:71`: "its review now waits for the operator's own ending whatever the autonomy".
   - `internal/dobbyengine/reinterpret.go:298`: the reconcile notice string, "except that the review now waits for the operator's own ending whatever …". This one reaches a reader at run time.
   - Neither mentions that the PR-Review session or an orch may end the review on the operator's word. Check any test that pins the notice string before rewording it.

2. **MEDIUM**, F2 fix (`prreview_session_test.go` `prSessionWorld`, `shuttlefake.ResumeEngine`, `envkit.LandingDeps`, the new wait rows): the set of stand-ins F2 names is only partly removed, and the fix adds some.
   - Removed: `fakePRSessions` and `noopSessions`.
   - Still hand-written for reed and creel:
     - `prSessionWorld.isLive`, which re-implements the liveness read production gets from `loomcli`'s `sessionLiveness` over reed.
     - The `MailAgent` closures in `prReviewTestDeps` and `newWaitWorld`.
     - The `SessionLive`, `RegisterSession` and `MailAgent` closures in `envkit`.
   - Newly added by this fix:
     - `shuttlefake.ResumeEngine`, a new fake capability of a provider engine.
     - The `MailAgent` closure returning "no such strand" in `AGateFailureWhoseMailFailsHaltsAndRecordsNoMailedHead`, a made-up creel error.
     - `envkit`, a kit, now builds its runner over `shuttlefake`, which `docs/research/no-fake-other-modules.md` says stands in for repository modules.
   - `PATTERN-test-stand-ins` says "No change adds a stand-in this rule forbids" and "An error of another module is provoked in the real module".
   - The fixer-report therefore cannot claim F2 as fixed as proposed. "Real reed and creel" is not met, so the remainder has to be stated, either as `fixed-differently` with these items named or as `deferred`.

**Sets that are fully covered**
- **F3:** the mail hint, the stencil, the `pr-continue` Long text and `docs/overview.md` all say the retry also ends the review. The other `pr-continue` mentions make no retry-only claim.
- **F7:** `prContinueVerb` is the only admission site.
- **F12:** every listed behaviour has a row.
- **F1:** the keys on the edited lines are written in full.

**Tool calls**
1. `git grep` for `pr-continue` with a bad exclude pathspec: it failed. I needed repo-wide sites, which my context lacked.
2. `git grep` for the ending wording, same bad pathspec: it failed.
3. `git grep` for the gate-mail and session wording, same bad pathspec: it failed.
4. `git diff main` over test files and testkit for added stand-ins: my context held only part of the diff. It succeeded.
5. `git grep` for `pr-continue` with a corrected pathspec: repo-wide F3 sites.
6. `git grep` for the ending, mail and session wording with a corrected pathspec: repo-wide F13 and F3 twins.
