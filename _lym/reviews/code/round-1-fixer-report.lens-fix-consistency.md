```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the change set against its surroundings.

1. **LOW**, changed text: `_lym/landing/summary.md`, the caller list under "The change" ("Every caller that classifies through this function follows").
   The list claims to cover every caller but omits `rejectionRetryKind`, whose answer for a missing local origin changes.
   Evidence: `internal/fabricengine/pushclassified.go:647` has `case errors.Is(err, gitrepo.ErrPushRetriesExhausted), gitexec.IsTransportFailure(err): return true, false`. Before the branch's change, such a failure fell to `default: return false, true`.

2. **LOW**, changed text: `_lym/landing/summary.md`, "outside fabric, `shedtransient.Class` marks the failure…" and `**Blast radius:** fabric, shed`.
   The text presents `shedtransient.Class` as the only effect outside fabric, but batten calls one of the listed fabric functions directly and also branches on the class.
   Evidence: `internal/battencli/wire.go:1180` calls `fabricengine.AddOptions{...}.RetryOriginReadInTransit("beam", …)`, and `internal/battencli/land.go:107` compares `shedtransient.Class(pullErr) == shedengine.TransientGitTransport`. `internal/beamcli/pr.go:492` also consumes `Class`.

I found nothing else stale: the doc comments, the package doc, the test headers and the `testtiming:keep` comment agree with the changed code.

Tool calls:
- One `grep` for twins of the "unmounted/missing/unreachable" wording across Go and markdown files, because my context held only the files the fixer had opened. The other evidence came from the caller grep and file spans already in context.
