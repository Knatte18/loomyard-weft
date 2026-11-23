```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change in the diff, find what calls, reads or describes it, and report anything the diff leaves wrong.

**What each change reaches, and whether it is still right**

1. `ReadPRReviewStanding` now returns satisfied when a satisfied answer stands at the head, even if a later ask at that head is still open or ended unanswered.
   - `loomcli/prcontinue.go` `prContinueUnderAutonomy` and `prContinueParentStep`: still right. Ending the review here is the intended effect.
   - `landingshed/prreview.go` `judgeParentAnswer`: still right, for the same reason.
   - `loomcli/parentreview.go` `parentReviewAskVerb` and its awaiting check: wrong. See F1.
   - `loomcli/doc.go:57`, which says the ask verb "refuses an ask that already awaits an answer at the head": wrong. See F1.
   - The `PRReviewAwaiting` constant's doc at `prreviewask.go:90`: mostly right. It does not say that a satisfied answer takes precedence. See F4.
   - `envkit.go:283` stub: not affected.
   - `contracts/orchrole/role-autonomy.md:48` and `orch-role.md:214`: still right.
   - `TestReadPRReviewStanding`: covered.
2. The stencil line added in `landing-block-pr-review-autonomous.md:20`.
   - The next line, 21, "It ends the review": wrong. See F2.
   - The await-outcome rule at lines 23–24: wrong. See F3.
3. The `doc.go` addition at `landingshed/doc.go:22-23`: consistent with the code.
4. Commit messages and PATTERN rules:
   - The `Root-Cause:` trailer is present and names the defect.
   - There is no `Pattern-Departure:` trailer and none is needed.
   - No declared PATTERN changes exist, and no PATTERN entry is broken.

**Findings**

- **F1 — MEDIUM, design.** `internal/loomcli/parentreview.go:326-335` and `internal/loomcli/doc.go:57`.
  - The ask verb now posts a new ask while an earlier one is still open at the head, whenever a satisfied answer stands before it.
  - Cause: the standing comes back satisfied, not awaiting, so the "refuses an ask that already awaits an answer at the head" guard never fires.
  - The same promise is also broken in the verb's own doc comment at `parentreview.go`, "It refuses an ask still awaiting its answer at the head".
  - Evidence: `prreviewask.go:112-123` returns satisfied before the awaiting check at line 130.

- **F2 — LOW, consistency.** `contracts/stencils/landing/landing-block-pr-review-autonomous.md:20-21`.
  - The new line sits between the `pr-continue` rule (line 19) and "It ends the review" (line 21).
  - "It" now reads as the satisfied answer rather than `lym loom pr-continue`.

- **F3 — MEDIUM, consistency.** `contracts/stencils/landing/landing-block-pr-review-autonomous.md:23-24`.
  - That rule still says to ask again whenever `await` reports a timeout, no-parent or not-sent end.
  - `await` reports only the latest ask: `parentReviewAwaitVerb` reads it through `ExpireParentAsk`.
  - So with a satisfied answer standing and a later ask timed out, the stencil sends the session to ask again rather than run `pr-continue`.
  - That contradicts the new line 20 and can keep the ask loop from the board entry going.

- **F4 — NIT, consistency.** `internal/landingshed/prreviewask.go:90`.
  - The `PRReviewAwaiting` doc, "an ask sent at the head that the parent has not answered, before its deadline", now also needs "and no satisfied answer stands at the head".
  - As written, it describes cases where the function returns satisfied instead.

**Tool calls**
- grep of the contracts and the package docs for "satisfied answer" and "latest ask": my context lacked the other docs and stencils that describe the standing.
- sed of `role-autonomy.md`, the `loomcli/doc.go` passages, and grep of `parentreview_test.go`: I needed the exact wording of the ask-verb and `pr-continue` docs the change reaches.
