```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change in the diff, find what calls, reads or describes it, and report whatever the diff leaves wrong.

**Affected set per change**

1. **`AnswerParentAsk` renamed to `AnswerParentAskOrUnreached` with a new `unreachedTag` parameter.**
   - Production caller `internal/loomcli/parentreview.go:235`: covered.
   - Test callers in burlerengine, driftengine (`judgeparent_test.go`, `verify_test.go`), landingshed, loomcli (`commitrecords_test.go`, `parentreview_test.go`) and shedadapters (`bouncer_parent_test.go`, `parentseat_test.go`): call sites covered, but their failure messages still name the old function (finding 3).
   - The `ParentAnswerRefusal` doc comment: covered.
   - The test comment at `internal/burlerengine/parentreview_test.go:609`: not covered (finding 3).
   - `_lym/landing/summary.md`: not covered (finding 2).
   - No `.md` or contract outside `_lym` names `AnswerParentAsk`: covered.
2. **An ended no-parent or undelivered ask under the tag is now answerable.**
   - `burlerengine/doc.go` ask lifecycle: covered.
   - `loomcli/doc.go`, the answer verb's doc comment and its `Long` help: covered.
   - `parentAnswerRefusalText` and `parentReviewWayPRReview`: still right for the cases still refused (cleared, timed out, superseded).
   - `ReadPRReviewStanding` (`landingshed/prreviewask.go:96-127`): reads `done` at the ask's `Digest`, so the answer stands at the head the ask named. Covered, and its doc comment is still accurate.
   - `PRReview.judgeParentAnswer` (`landingshed/prreview.go:968-990`): consistent.
   - `askParentReview` doc (`prreview.go:375`): consistent.
   - `orch-role.md:202-214`: still accurate, since it defers to `role-autonomy.md`.
   - `role-autonomy.md`: changed, but not consistent with the stencil for the no-parent case (finding 1).
3. **The autonomous landing stencil gains the "await first" step.**
   - Lines 22-31: no earlier rule is dropped or weakened; "ask again" is narrowed to "after `await` still reports ended".
   - The pairing with `role-autonomy.md`: not consistent for no-parent (finding 1).
4. **Declared PATTERN changes:** the directive declares none, so there is nothing to land.
5. **Commit messages** (4e863bbab, b696b4cca): both match the diff. There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed: the edits to existing tests only rename the call and add `""`.

**Findings**

1. **MEDIUM, consistency.** Location: `contracts/orchrole/role-autonomy.md:48-49` together with `contracts/stencils/landing/landing-block-pr-review-autonomous.md:24-31`.
   - The orch contract says to answer "when the run's PR-Review session mails you an ask that ended no-parent".
   - The stencil has the session mail its parent only after an undelivered end (`lym creel post --parent` with the reason and ask number). After a no-parent end it only ends its turn.
   - So no path tells the parent the number of a no-parent ask. The no-parent half of the new contract line, and the stencil's "your parent may have answered that ask" after a no-parent end, rest on a mail that is never sent.
   - Evidence: `landing-block-pr-review-autonomous.md` lines 24-27 (no mail on no-parent) against lines 26-28 (mail on undelivered only), and `role-autonomy.md` lines 48-49.
2. **LOW, consistency.** Location: `_lym/landing/summary.md`, "What changes", third sub-bullet.
   - The summary says "`AnswerParentAsk` keeps its behavior and delegates with no tag, so review rounds and the drift judge are unchanged."
   - Commit b696b4cca deleted that wrapper. No `AnswerParentAsk` function exists at HEAD, and every caller now calls `AnswerParentAskOrUnreached` directly.
   - Evidence: the grep for non-test `AnswerParentAsk` returned nothing; the diff pieces show each call site rewritten; the b696b4cca subject reads "drop the test-only AnswerParentAsk wrapper".
3. **NIT, consistency.** Location: `internal/burlerengine/parentreview_test.go:609` and the test failure messages.
   - The doc comment "TestReadParentReport verifies ReadParentReport returns what AnswerParentAsk wrote" names a function that no longer exists.
   - The `t.Fatalf`/`t.Errorf` messages "AnswerParentAsk() error = …" left the diff's call sites naming the old function:
     - `parentreview_test.go` lines 317, 325, 328, 361, 365, 424, 567 and 731
     - `driftengine/judgeparent_test.go` lines 185 and 268
     - `driftengine/verify_test.go:2075`
     - `loomcli/commitrecords_test.go:350`
     - `loomcli/parentreview_test.go:637`
     - `shedadapters/bouncer_parent_test.go` lines 73 and 155
     - `shedadapters/parentseat_test.go:144`
   - Evidence: the grep output listing these lines.

**Tool calls**
- Grepped the repository's `.md` and `.go` files outside `_lym` for the old function name, `no-parent` and end-state wording: the context lacked the docs, contracts and test messages beyond the diff.
- Read `parentReviewCmd`'s help text (`loomcli/parentreview.go:440-520`), grepped contracts for `pr-review request` and the parent-review verbs, and grepped `landingshed/prreview.go` for ask-state handling: the context lacked the help text and the other stencils naming the verbs.
- Read `orch-role.md:198-235` and `landingshed/prreview.go` lines 365-400 and 960-1000: the context lacked the orch role's PR-Review section and the row's handling of ask states.
