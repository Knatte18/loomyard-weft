```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The new "asks and does not gate" wording agrees with the text around it:
- `contracts/orchrole/orch-role.md` says "the operator ends the review" and tells the orch to answer the PR-Review session.
- `internal/landingshed/deps.go` comments on `ParentReviewWanted` and `AskParent` make no claim about waiting or about a failed record write.
- `doc.go` and the `askParentReview` doc comment now both list the failed record write as logged, and both say the next entry asks again.
- The `prreview.go` code and the `PRReview` type doc ("asks the parent orch ... before it waits") match.
- `docs/overview.md` carries the same "asks and does not gate" statement as `doc.go`.

Two things I noted and did not rate as findings:
- F2's review fix also asked for a test row for a record write that fails after a successful post. The commit adds none. That is a gap against the review, not a disagreement between the changed text and its siblings.
- F1 left the board-entry premise ("waits at PR-Review") unresolved in the repo. The reviewer's alternative asked for a `lym loom decision add` record and rewording. The rewording was done, and the decision record was not.

Tool calls (1):
- `sed` of `internal/landingshed/deps.go` lines 380-395 and `contracts/orchrole/orch-role.md` lines 196-206. The fork context did not hold `deps.go`, and it held only part of the orch-role contract.
