```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for F1, F2 and F5 and check the fix covers every member.

**Findings:** no findings.

**Unchecked:** F3 and F4 are test-coverage findings, not set premises, so I ran no search for them. The F4 row's counts and Warn text were not checked against the suite's result.

- **F1 and F5, callers:** `haltOnEndedSession` now has one caller, `prreview.go:738` (after `judge`). The `!polled` branch returns nil, so no site is left on the old rule.
- **F1, prose:** no text outside the changed lines states the old rule. The matches are the comments at `prreview.go:618`, `:663` and `:747`, which all read consistently with the fix. I found no `doc.go`, spec or other markdown that repeats the "no status" check besides `_lym/landing/summary.md`, which the fix updated.
- **F2, `SessionLive` readers:** the other PR-Review readers are `prsession.go:80` and `:249` in `ensureSession`, and `samplersession.go:203`, `:248` and `:388` in the sampler row. The review said to leave the `ensureSession` path unchanged. The sampler row is outside this review's scope.
- **F5, `wait` call sites:** the production sites are `prreview.go:306`, `:346` and `:371`, plus the test helper at `prreview_wait_test.go:1155`. With the `ReviewEnded` guard dropped, `:306` and the test helper pass `false`, so the guard stays correct there. `:346` passes `keepsSession`, false only for a merged pull request. `:371` passes `true` on a fresh or open review, where `ReviewEnded` is false.

**Tool calls (3 of 6):**
1. `grep SessionLive` across `*.go`: to list every reader of the liveness seam, which my context lacked outside `landingshed`.
2. `grep` for the old and new wording across `*.go` and `*.md`: to find prose that might still state the old rule.
3. `grep` for `wait(`, `keepsSession`, `haltOnEndedSession` and `liveWarned`: to list every `wait` call site and flag use.
