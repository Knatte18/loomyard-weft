# Let the parent answer a PR-Review ask that never reached it

A PR-Review session's `lym loom parent-review ask` can end `no-parent` or `undelivered` while the parent orch is live.
`parent-review answer` then refused that ask ("already no-parent"), and `pr-continue` refused with "no satisfied answer from your parent stands".
A review whose parent was satisfied at an unchanged head could not end.

`parent-review answer` now answers a latest PR-Review ask that ended `no-parent` or `undelivered`.
The ask becomes `done`, and the answer stands at the head the ask named, so `pr-continue` accepts it.
This change does not alter how the ask finds the orch's pane; it removes the dead end the failed lookup left.

Closes #206

## Design decisions

- `burlerengine.AnswerParentAsk` is renamed `AnswerParentAskOrUnreached` and takes an `unreachedTag`.
  A latest ask made under that tag, in state `no-parent` or `undelivered` and not cleared, is answered instead of refused.
  The deadline check is skipped for such an ask, since its request never reached the parent.
- An empty tag, another tag, a cleared ask, a `timeout` ask and a non-latest ask are all still refused.
  Every caller other than `parent-review answer` passes `""`, so their behaviour is unchanged.
- `parent-review answer` passes `landingshed.PRReviewAskLineTag`, so the exception covers PR-Review asks only.
  Code-review and other parent asks keep refusing an ended ask.
- The orch role contract (`role-autonomy.md`) and the autonomous PR-Review landing block now say so.
  After a `no-parent` or `undelivered` end, the PR-Review session runs `await` on its next mail before asking again, since an answer may already stand.
- Help text and package docs for `loomcli` and `burlerengine` describe the new move out of an end state.

## To check by hand

- The orch contract line in `contracts/orchrole/role-autonomy.md` says an ask ended `no-parent` or `undelivered` is one "its PR-Review session mails you".
  Check that the PR-Review session does mail the parent about an unsent `no-parent` ask, as the landing block tells it to for an undelivered one;
  the contract may name mail the session never sends.
- Not run: the live case from #206, a PR-Review ask recorded `no-parent` while the orch is live in another tmux window, answered with `parent-review answer` and then `pr-continue`.

## Merge danger

**Door:** two-way
**Blast radius:** burlerengine
