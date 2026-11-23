```yaml
lens: fix-code-correctness
outcome: reported
```

**Lens:** fix-code-correctness. I traced F1, F2, F3, F4, F5, F7 and F8 on concrete inputs. I found nothing wrong in the logic, and one gap in what the F2 retry bound covers.

**Findings**

1. LOW, `internal/battencli/arm.go:519` (`ArmAt`) with `internal/battencli/arm.go:348`.
   - `ArmAt` builds `&battenCLI{}`, so `reconcileOptions` is zero. For verb `run` it then sets `c.readopt = hubReadoptSeams(location, c.reconcileOptions)`, so that receiver's gone-pair reconcile still makes one `ls-remote` attempt.
   - Evidence: `grep` finds no non-test caller of `battencli.ArmAt` (`internal/loomcli/arm.go:507` is a separate function), so nothing in production hits this today. It would silently skip the bound if a caller appeared.

**Traced, no defect**

- **F1:**
  - A transport `*GitError` from `ReadParentTips(true)` after the merge: `transientFailure` returns a non-nil error, so `mergeOnGitHub` returns `&ending{err: terr}` and the driver re-steps.
  - A non-transport error still reaches the halt.
  - The `merged` flag set in `onCall("Merge")` makes only the post-merge read fail, and earlier reads go through the saved `readTips`.
  - The transient row would fail with the old halt in place.
- **F3:**
  - The 413 rows contain `rpc failed; http 4` or `returned error: 4`, so they return false.
  - The existing true rows (502, 503, 504, HTTP/2, `Recv failure`) match neither guard string.
  - The `Timeout != 0` early return stays ahead of the guard, so a timed-out command is still true.
- **F8:** the sideband and early-EOF phrases are now separate rows, so deleting either phrase alone fails.
- **F2:**
  - Every `reconcileRun`, `reconcileGoneRun`, `hubReadoptSeams`, `tearDownLeftovers` and `reconcileEnded` caller is updated, and I found none missed.
  - The new battencli test sets `Sleep` to restore the origin URLs. With zero options it would see `Unreachable` and fail, so it catches the defect.
- **F7:**
  - `retryTransportPush(nil)` returns nil.
  - A non-transport error returns at once.
  - A persistent transport failure makes four pushes in all and returns the last error, which `classifyFailure` still records as `PushFailureTransport`.
  - The new test would fail without the retry, because the first push would be recorded and the tip would not match.
  - The existing never-pushed row was switched to a no-op backoff, so it no longer sleeps for real.
- **F4 and F5:**
  - `errors.Is(err, tt.wantRunErr)` is correct when both are nil and when either one is set.
  - `waitMovesTo` takes effect only after a `wait:` call, so a wait moved after the recheck would leave the status unmoved at the recheck, record a re-step, and fail the row.
  - The context-cancelled wait row returns before `holdPause` and the recheck, so it records nothing, as the row expects.
  - `TestGoDriver_Wait` is deterministic: the `time.Hour` delay with an ended context returns through `ctx.Done`, and the 1ns timer fires.

**Tool calls**
- `sed arm.go` and `grep reconcileOptions|armAt`: my context lacked the `ArmAt` constructor at `arm.go:520`, which builds a receiver with zero options.
- `grep ArmAt(`: to find whether any production caller passes verb `run` through that receiver.
