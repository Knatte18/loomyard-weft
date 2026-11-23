---
round: 2
focus:
  - "internal/landingshed/finalize_pr.go, the ReadParentTips(true) call after the pull request merges: when the fetch fails with a transient network error, does Finalize retry it with backoff before halting with LandingStopPrefix, and does the halt reason name the failing fetch?"
  - "Run-start and catch-up origin reconciles at internal/battencli/resumereconcile.go, internal/battencli/readopt.go and internal/beamcli/dobbydeps.go, which build a fabricengine ReconcileOptions: does each pass a production RetryBound and RetryInterval so one failed ls-remote is retried with backoff before the run halts or a hand pass is refused, and can tests set the bound short?"
  - "internal/gitexec/transport.go IsTransportFailure: does a stderr holding an HTTP 4xx refusal (such as RPC failed with HTTP 413) followed by a sideband disconnect or Send failure line still classify as a permanent refusal rather than a transport failure?"
  - "internal/sheddriver/drive.go goDriver.wait and settleStop: do tests cover the real-clock branch returning the context's error, the exit on a wait error with no re-step or hand-back, and the wait running before the pause hold and status recheck?"
  - "internal/sheddriver/doc.go and pattern/PATTERN-shed-driver.md: do the optional-seam lists name Deps.Wait and say what an unwired one does?"
  - "internal/fabricengine weft push paths that call gitexec.IsTransportFailure (pushanchored.go, pushclassified.go): is a transport failure on a board or records-branch push retried with backoff before it is reported owed, and does a test cover one failure then success?"
  - "internal/gitexec/transport_test.go TestIsTransportFailure: does each new transport phrase have a row of its own, so deleting one phrase fails a test?"
---

Each entry restates its sites so it stands alone.
