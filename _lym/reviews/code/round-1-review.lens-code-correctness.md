```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- **`orchcli.PostAsCaller` with `strandClaim=""` (the row's ask).** `claimsNonOrchStrand("")` is false, so it calls `postAsProgram` through `notifyClientsOf(location, "", program).program`, which is `NewScopedProgramClient(location, "", program)`. That is the same client the old `PostAsProgram` call built, and the same `noticeError` wrapping. Correct.
- **`PostAsCaller` with `strandClaim="mo:x:pr-reviewer"`, called from inside that strand's pane.** It uses the strand client, and creel's `resolveSender` finds the pane, sees the claim equals `pane.Name` and the program is empty, so it returns `StrandSender`. The post is delivered signed as the strand (`internal/creelengine/identity.go:66-72`). Correct.
- **`PostAsCaller` with the same claim, called from a process in no pane.** Creel refuses with `ErrStrandClaimOutsidePane`, which goes to the `postAsProgram` fallback. With no pane, the program claim is allowed (`identity.go:87-95`). Correct.
- **`PostAsCaller` where the strand post fails for another reason, such as an unreadable pane listing or a daemon error.** `noticeError(line, err)` is returned and nothing falls back. The caller ends the ask `undelivered` with that error as its reason. Correct, and the error is not swallowed.
- **`PostAsCaller` with a claim that names an orch, or a malformed name.** It goes straight to the program client, which is allowed from an orch pane. Correct.
- **`PostAsCaller` where the `clients.strand` builder itself fails.** The `noticeError` is returned with no fallback, which matches what `notifyPrime` did before. Correct.
- **`notifyPrime` refactored onto `postSigned`.**
  - `orch()` error: returns `(false, err)`, unchanged.
  - Not recorded: returns `(false, nil)`, unchanged.
  - Strand success: `(true, nil)`.
  - Strand refusal with a non-sentinel error: `(false, wrapped)`.
  - Sentinel refusal followed by program success: `(true, nil)`.
  - Program builder or post failure: `(false, noticeError)`, the same wrapping as the old inline code.

  Behaviour is identical. Correct.
- **`postNotice` on `notifyClientsOf(location, slug, program)`.** It builds the same strand and scoped-program closures as before. Correct.
- **`loomcli.prReviewAsker(l, rec, strandClaim)`.** The row passes `""` (`landingdeps.go:478`), and `parentReviewState.preRun` passes `os.Getenv(agentname.StrandNameEnv)` (`parentreview.go:408`). With an empty env var, from the operator's shell, the post signs as the program from no pane, which creel allows. Correct.
- **`landingshed.PRReviewAsker.Ask` when the post fails.** `EndParentAsk(..., ParentAskUndelivered, "the request was not delivered: <err>")` runs, and it returns the ended ask and `endErr`. Resolve error and no strand still record `no-parent` before any post (`prreviewask.go:52-54`). Correct.
- **`ReadPRReviewStanding` with an undelivered ask at head.** That state falls outside both cases, so the result is `PRReviewAskAgain`. Correct.
- **`shedadapters.parentAsker.post` when the post fails.** The ask is set `undelivered` with the reason. `send`'s next-round logic (`parentseat.go:171-180`) re-asks only on `NotSent` or `Done`-with-findings, so `undelivered` behaves as `no-parent` did before, with no new loop. Correct.
- **`driftengine.JudgeParentSeat.postJudgeAsk` when the post fails.** The record is set `undelivered`, and the returned ask has `State=Undelivered` with the reason.
  - If `EndParentAsk` itself fails, the record stays `pending-post` while the in-memory ask says `undelivered`. This was the same for `no-parent` before the diff, so the diff introduces nothing new.
- **`DriftVerify.endedUnanswered` with an undelivered latest ask.**
  - Not cleared: halts under autonomy, as `no-parent` does.
  - Cleared: returns false, so the resume asks again.

  Correct.
- **`parentReviewAwaitVerb` with an undelivered ended ask.** `Ended()` is true, and the result carries `reason`. Correct.
- **`ParentAskState.Ended()` on an older binary reading `"undelivered"`.** An unknown state counts as ended, so it never moves. Correct.
- **Test helper `startNoticeOrch(t, h, others...)`.** It appends the others after the receiver, and the existing callers pass none. Correct.
- **Test helper `postedEntries` / `postedSubjects`.** `postedSubjects` gives the same result as before. Correct.

**Findings**

None.

**Tool calls**

None. The inherited context held every definition and call site the traces needed.
