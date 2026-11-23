```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I looked for twins of the changed text that the round left as they were. Two findings, both MEDIUM and both about missed twins.

**MEDIUM: ParentAlign's parent fetch still halts on a transport failure (F1 twin)**
- Changed text: `internal/landingshed/finalize_pr.go`, `mergeOnGitHub`. It now returns `transientFailure` for a failed `ReadParentTips(true)` after the merge.
- The same fetch, `ReadParentTips(true)`, runs in `internal/landingshed/parentalign.go` `mergeTip`. It wraps the error with `fmt.Errorf`, with no `transientFailure` and no `ending{err}`. The error path at the call site may differ from Finalize's, so a transport failure there may not be re-stepped. I read only `mergeTip` itself, so this is unconfirmed.
- Finalize's own pre-merge `ReadParentTips(true)` is a separate site I did not check either.
- The review named only the post-merge site, so this is a sibling left behind rather than a miss against the finding.

**MEDIUM: other run-start origin reconciles and the hand-pass catch-up still make one attempt (F2 twin)**
- The new `fabricengine.RunStartReconcileOptions` doc says it serves "a run's start, resume or leftover teardown". Other sites still pass a zero `ReconcileOptions{}`:
  - `internal/fabriccli/remove.go:226` (run scope)
  - `internal/loomcli/landingdeps.go:874` (run scope)
  - `internal/orchcli/start.go:385`
  - `internal/cleanupcli/reconcile.go:36`
  - `internal/battencli/cancelcmd.go:371` and `internal/battencli/wire.go:1291`
- The hand pass's catch-up fetches are untouched. #203's refusal is "origin is unreachable" from `internal/battencli/catchup.go:64`, and it comes from `fetchRunForCatchUp` and `pullBoltForCatchUp`, which map any fetch failure to `ErrOriginUnreachable` with no retry. These are not reconcile calls, so the F2 fix cannot cover them.
- Not every site is a run-start (the cancel and judge-only ones are probably intentional). The doc wording and the catch-up gap should still be reconciled.

No further inconsistencies found in the F3 transport guard, the F6 `Wait` docs (they agree with the `Deps.Wait` comment) or the F7 doc comment.

Tool calls:
1. Bash (`sed`, `grep`): read `parentalign.go` `mergeTip`, listed every remaining `ReconcileOptions{}` site, and checked `gitexec/doc.go` for phrase references. My context lacked all three.
