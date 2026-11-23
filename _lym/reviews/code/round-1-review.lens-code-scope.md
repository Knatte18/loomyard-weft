```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each of the diff's changes to the goal points of the board entry.

**Decisions.** The decision record holds no plan, and its `## Decisions` section is empty. It has no `Added after Discussion` entries. The goal points come from the board entry under `## Goal`, which quotes issue #200:
- **G1:** a re-ask from the PR-Review session is delivered, signed by a sender allowed to claim it or sent through the row.
- **G2:** a refused delivery is recorded as a delivery failure, not as no-parent.

**Per change:**
- **C1** traces to G1. It adds `orchcli.PostAsCaller`, `postSigned` and `notifyClientsOf`, and moves `notifyPrime` onto `postSigned` (`internal/orchcli/notify.go:137-182`, `:264-290`). It also updates `internal/orchcli/doc.go:113-115`, which C1 forces. This is the signing path that lets the session post as its own strand.
- **C2** traces to G1. It adds the `prReviewAsker` strand claim (`internal/loomcli/landingdeps.go:498-516`). The row passes `""` (`:478`) and `parent-review ask` passes the session's strand (`internal/loomcli/parentreview.go:408`). The `loomcli/doc.go:50` line and the `ask` Long help (`parentreview.go:514-516`) are forced by C2.
- **C3** traces to G2. It adds the `ParentAskUndelivered` state (`internal/burlerengine/parentreview.go:38-39`). The `burlerengine/doc.go` state list is forced by C3.
- **C4** traces to G2. A failed post ends `undelivered` in three places: the PR-Review asker (`internal/landingshed/prreviewask.go:64`), the review seat (`internal/shedadapters/parentseat.go:226`) and the drift judge seat (`internal/driftengine/judgeparent.go:207`). G2 is stated generally, so all three seats that record a failed post fall under it. The doc comments at `prreviewask.go:44,101`, `parentseat.go:221` and `judgeparent.go:202` are forced by C4.
- **C5** is forced by C4. Drift-Verify's `endedUnanswered` (`internal/driftengine/verify.go:887`) and `driftengine/doc.go:98` keep the autonomy halt that a failed judge post triggered while it was recorded `no-parent`.
- **C6** traces to G2, as a reader of the new state. `parent-review await` answers an undelivered ask's reason (`internal/loomcli/parentreview.go:254`, `:272`, `:487-488`).
- **C7** traces to G2, which C3 forces. The autonomous PR-Review stencil gains three lines telling the session that undelivered is not a down parent (`contracts/stencils/landing/landing-block-pr-review-autonomous.md:26-28`). This is the misreading the issue's stall came from.
- **C8** is forced by C1 and C2. `contracts/orchrole/orch-role.md:202-203` says the `[pr-review request]` mail may now come from the PR-Review session's strand as well as `<shortname>:LOOM`.
- **C9** is forced by C3, C4 and C6. It adds failed-post rows that expect `undelivered` in:
  - `TestPRReviewAsker_Ask`
  - `TestParentAgreementGate`
  - `TestBurlerReviewProducer_Call_ParentAskThatDoesNotGoOut`
  - `TestDriftVerify` (its `Undelivered` row)
  - `TestParentReviewAwaitVerb`
- **C10** is forced by C1 and C2. It adds `prreviewask_integration_test.go` and extends the `startNoticeOrch` and `postedEntries` helpers in `notifystamp_integration_test.go`.

**Per decision:**
- **G1** is built by C1 and C2, with C8 and C10 forced by it.
- **G2** is built by C3, C4, C6 and C7, with C5 and C9 forced by it.

No finding: every change traces to G1 or G2 or is forced by a traced change, and both goal points have changes that build them. No commit of the range carries a `Pattern-Departure:` trailer, and none is needed.

Tool calls: none; my context already held everything this procedure needed.
