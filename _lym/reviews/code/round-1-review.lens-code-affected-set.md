```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what else reads or describes it, and flag anything the diff leaves wrong.

**Per change, the set found, covered or not**

1. **New state `ParentAskUndelivered` (`internal/burlerengine/parentreview.go:38-39`)**
   - `Ended()` at `parentreview.go:46` treats any state except pending-post and open as ended: covered.
   - The `burlerengine` doc.go state list was updated: covered.
   - The `RecordParentAsk` doc at `parentreview.go:203` lists only no-parent and not-sent for an ask that never goes out. That is still right, because undelivered is set only through `EndParentAsk`.
   - The `EndParentAsk` doc at `:240` names no states: covered.
   - Consumers:
     - `ReadPRReviewStanding` (`landingshed/prreviewask.go:112-124`) falls to ask-again, and its doc was updated: covered.
     - `parentAsker.shouldAsk` (`shedadapters/parentseat.go:171-180`) re-asks only after not-sent, so undelivered behaves as no-parent did before: unchanged, covered.
     - `endedUnanswered` (`driftengine/verify.go:887`) and the `driftengine` doc.go: covered.
     - The await verb (`loomcli/parentreview.go:272`), its Long text and the `TestParentReviewAwaitVerb` row: covered.
     - `bouncer.go:1163` and `burlerengine/parentseat.go:80` test only for done: unaffected.
   - Persisted `asks.json`: an older binary counts the unknown state as ended (`Ended()` doc), and its standing becomes ask-again. Covered.
2. **Failed post ends undelivered, in all three seats** (`landingshed/prreviewask.go:64`, `shedadapters/parentseat.go:226`, `driftengine/judgeparent.go:207`)
   - The doc comments at `prreviewask.go:44`, `parentseat.go:221` and `judgeparent.go:202` were updated.
   - The tests `TestPRReviewAsker_Ask`, `TestBurlerReviewProducer_Call_ParentAskThatDoesNotGoOut`, `TestParentAgreementGate` and the `TestDriftVerify` rows were updated.
   - The no-parent docs at `prreviewask.go:45`, `parentseat.go:186` and `judgeparent.go:167` now speak only of resolve failures, which is right.
   - Covered.
3. **`prReviewAsker` signature gains `strandClaim` (`loomcli/landingdeps.go:502`)**
   - Callers: `landingdeps.go:478` passes `""` and `parentreview.go:408` passes `strand`. Both updated.
   - Docs: `loomcli/doc.go:50` and the parent-review ask Long text were updated.
   - Test: `prreviewask_integration_test.go`.
   - Covered.
4. **New `orchcli.PostAsCaller`, and `postSigned` and `notifyClientsOf` split out of `notifyPrime` and `postNotice`**
   - `postNotice` and `notifyPrime` callers keep the same behaviour, since the program fallback now goes through `postAsProgram` and gives the same `noticeError` wrapping.
   - `orchcli/doc.go` was updated.
   - `PostAsProgram` remains at `parentseatwiring.go:103` and `driftwiring.go:179`. Both run in the driver process, not a session pane, and their docs (`parentseatwiring.go:90`, `driftwiring.go:173`) still say "signed as lym's loom program". Still correct.
   - `cli.go:189` (`notifyParent`) is unaffected.
   - Covered.
5. **`startNoticeOrch` signature and the `postedEntries` split (`notifystamp_integration_test.go`)**
   - Existing callers still compile with the variadic `others` and can ignore the returns, and `postedSubjects` is kept.
   - Covered.
6. **Orch role (`contracts/orchrole/orch-role.md:202-203`)**
   - Answer guidance at `orch-role.md:205-216` (answer the session the request names, or `parent-review answer` under autonomy) needs no sender change.
   - Covered.
7. **Autonomous PR-Review stencil (`landing-block-pr-review-autonomous.md:26-28`)**
   - The read-back list at line 22 ("timeout, no-parent or not-sent") leaves undelivered out, but the new paragraph handles it separately. Consistent.
   - The operator stencil (`landing-block-pr-review-operator.md:14`) does not deal with ask end states: not affected.
   - `docs/overview.md:1399-1405` describes only the row's ask, as before the diff, so no change is forced.
   - Covered.
8. **Rule documents**
   - No PATTERN changes are declared, so none are owed.
   - No new refusal is added, so `PATTERN-refusal-way-forward` does not apply.
   - Creel remains the only messaging path (`PATTERN-agent-addressing`).
   - `asks.json` is still read and written through `internal/state` (`PATTERN-tolerant-records`).
9. **Commit messages and trailers**
   - The `Root-Cause:` trailer on 878f0bc14 names the defect the diff removes: `PostAsProgram` from a non-orch pane, and `ParentAskNoParent` on a refused post. It holds.
   - No `Pattern-Departure:` or `Loosened-Check:` trailers are present or needed; the edited test rows tighten their checks rather than loosen them.

**Findings**

- **LOW · consistency** · `contracts/stencils/landing/landing-block-pr-review-autonomous.md:26`
  - The stencil tells the session "An undelivered end means creel refused the ask", but the code also ends a post undelivered when it fails without a creel refusal.
  - Evidence: `PostAsCaller` returns `noticeError` for a client-build or post failure (`internal/orchcli/notify.go:279-289`, `postAsProgram` at `:148-157`). Each seat ends any `Post` error undelivered, as the `burlerengine` const doc says at `internal/burlerengine/parentreview.go:38`: "creel refused or failed".
  - The instruction that follows (send the reason to the parent) still fits.
  - The parent-review ask Long text, "An ask creel refuses ends undelivered" (`internal/loomcli/parentreview.go` Long), has the same narrowing but is not false.
- **NIT · consistency** · commit 878f0bc14 message
  - The message says the stencil tells the session "to send the reason to its parent rather than wait for mail".
  - The stencil says to send the reason, "then end your turn and ask again on the next mail you receive", so the session does wait for mail (`landing-block-pr-review-autonomous.md:27-28`).
  - `_lym/landing/summary.md` states this correctly ("then waits for mail").

**Tool calls**
1. Grep for every "no-parent" and "not-sent" in non-test Go and `.md` files, to find doc comments and stencils naming the ask states beyond the diff's hunks.
2. Read the `RecordParentAsk` and `EndParentAsk` docs, the "loom program" signing docs in `loomcli`/`landingshed`, and the specs or docs naming `parent-review ask`, to check docs describing the state set and the signing.
3. Grep `docs/overview.md`, the operator PR-Review stencil and `orch-role.md` for the ask and answer wording, to judge whether the docs the diff did not edit still hold.
4. Read the `docs/overview.md` PR-Review ask passage and the `orch-role.md` answer context, to confirm neither is left stale.
