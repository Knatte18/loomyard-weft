```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1, F2 and F7 and check the fix covers every member.

**F2: reconcile sites.** The three sites the review named are covered, plus the leftover teardown in `dobbydeps.go`. The same search still finds other `ReconcileOrigin` callers that pass a zero `ReconcileOptions{}` and make one `ls-remote` attempt:
- **MEDIUM** `internal/loomcli/landingdeps.go:874`
  - What: the run-scope reconcile that a run's landing performs still makes one attempt, with `ReconcileOptions{}`.
  - Why it matters: it is a run-scope reconcile of a live run, and one transport failure there reaches the parent with no backoff.
  - Evidence: the file still builds the call as `ReconcileOrigin(prime, ReconcileScope{Run: slug}, ..., fabricengine.ReconcileOptions{})`, and `RunStartReconcileOptions` is not used there.
  - Not checked: I did not read the surrounding code, so I can't say whether the landing treats an `Unreachable` report in some other way.
- **LOW** `internal/orchcli/start.go:385`: this is a hub-scope reconcile at orch start, still on zero options. It may be out of scope for "run start".
- **LOW** `internal/battencli/wire.go:1291`: a judge-only run-scope reconcile on zero options.
- **LOW** `internal/battencli/cancelcmd.go:371`: a hub-scope reconcile on zero options.
- **LOW** `internal/battencli/cancelcmd.go:204`: the fixer passed zero options deliberately.
- **LOW** `internal/loomcli/ownercheck.go:65`: a judge-only reconcile on zero options.
- **LOW** `internal/fabriccli/remove.go:226` and `internal/cleanupcli/reconcile.go:36`: both on zero options. These are operator-run verbs, so they are probably intentionally single-attempt.

**F2: catch-up fetches.** `internal/battencli/catchup.go` turns any fetch failure into `ErrOriginUnreachable` with no retry. This covers `fetchRunForCatchUp`, `pullBoltForCatchUp`, `registerThisMachine` and the `arm.go:638-642` fetches, and it is the source of the refused hand pass the review cites at `catchup.go:66`.
- **MEDIUM** `internal/battencli/catchup.go:216-229`
  - What: the fix leaves the catch-up fetch path with a single attempt.
  - Evidence: the `ls-remote` reconcile options do not reach `FetchPairTips` or `gitrepo.Fetch`, so #203's refused hand pass can still occur.
  - Note: the review's own fix key named only the reconcile sites, so this may be outside what was asked.

**F1: fetch sites.** The fix covers the post-merge fetch at `finalize_pr.go:526`. The other `ReadParentTips(true)` sites are `finalize.go:503` and `parentalign.go:236`. Their failures go through the existing paths, and I did not check whether those paths treat a transient failure as a re-step. Treat this as unchecked.

**F7: push sites.** `retryTransportPush` wraps only the main `job.push()` at `pushclassified.go:155`. These push sites do not retry a transport failure:
- **LOW** `pushclassified.go:390`: `return false, j.push()` in the retry path of a rejected push.
- **LOW** `pushclassified.go:934` and `:1019`: `repo.PushRebaseFree`.
- **LOW** `takeoverwrite.go:198`: `PushRebaseFree`.

For the first of these, `rejectionRetryKind` treats a transport failure as not retryable, so it is recorded at once. I did not read `:934`, `:1019` or `takeoverwrite.go:198`, so I don't know whether each is a retry path or a separate push.

**F3: phrases.** I made no tool call for it; my context shows the guard covers both phrases named in the review.

Unchecked: F4, F5, F6 and F8, because they are not set-shaped findings. That includes whether another production `Deps` builder besides `battencli` and `loomcli` leaves `Wait` unwired.

Tool calls made (3 of 6):
1. `grep -rn "ReconcileOrigin("`: the F2 caller set. My context held only the call sites the fixer grepped before editing.
2. `grep ReadParentTips(true)|.Fetch()|transientFailure(` in `landingshed`: the F1 sibling sites. My context lacked a full list of these.
3. `grep PushRebaseFree|.push()|retryTransportPush`: the F7 push-site set. My context lacked a full list of these.
