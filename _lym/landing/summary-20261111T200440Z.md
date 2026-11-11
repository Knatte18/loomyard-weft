# Parent-review ask record, deadline keys, and the answer and await verbs

This change lays the groundwork for a Discussion-Review round that asks the run's parent orch to review the decision record.
It adds the config keys, the per-run ask record, the capped deadline and the two verbs the parent and the review chair use.
Nothing posts an ask yet: the round that records and posts asks, and the chair stencil that runs the await, come in `discussion-review-parent-seat`.

## Config keys

`loom.yaml` gains two keys, both category `none`, so a task's `run_settings` sets them per run:

- `discussion_review_ask_parent` (default `false`) turns the parent review on.
- `discussion_review_parent_timeout_min` (default `30`) sets the minutes from an ask to its deadline before the cap.
  Config validation refuses `0` or a negative value, naming the key.

`loomengine.CapParentReviewWait` caps that wait the way `CapAssistTimeout` caps the assistant.
The chair's timeout (`review_timeout_min`, or shuttle's `run_timeout_min` when that is `0`) is reduced by
shuttle's start allowance, one chair keep-warm await cap and `ParentConsolidationMargin` (5 minutes).
The wait is the lesser of the key and that room.
A capped wait logs one Warn naming both keys.
A room of zero or less returns a zero wait and a reason telling the operator to raise `review_timeout_min`;
the caller then records the ask `not-sent`.
The chair-timeout rule is now one helper that both caps share.

## The ask record

`burlerengine` keeps each run's asks in `.lym/parent-review/asks.json`, never tracked.
The directory is named by `lymdirs.ParentReviewDirName`, and the lock path comes from `lock.TrackedLockPath`.
The record is written through `internal/state`, so a reader built from older code keeps the keys it does not know.
Each ask has a number, its round, the orch strand it went to, its ask time and deadline, and a reason or report path.
Its state moves only forward: from `pending-post` to `open`, then from either one to `done`, `timeout`, `no-parent` or `not-sent`.
Every move happens under the one lock:

- `RecordParentAsk` adds the next numbered ask, and `MarkParentAskPosted` moves it from `pending-post` to `open`.
- `AnswerParentAsk` writes the parent's YAML report (`ask-<n>.report.yaml`, with findings numbered `P1`, `P2`, …) and sets the latest ask `done`.
  An active ask at or past its deadline is set `timeout` instead, and the answer is refused.
  Every refusal is a `ParentAnswerRefusal` naming its kind.
- `ExpireParentAsk` sets the latest active ask `timeout` once its deadline has passed.

Because every writer takes the same lock, an answer and a timeout racing each other leave exactly one end state.

## `lym loom parent-review answer [<slug>] --ask <n>`

The parent answers with exactly one of `--satisfied` or `--findings-file <path>`.
The findings file is YAML holding a `findings` list.
Each item needs a `change` and may have a `severity` and `class` using the review format's words.
The verb refuses a file with no item, an item with no `change`, or a severity or class outside those words, and each refusal names the file's shape.
Three callers are admitted:

- the operator's own unnamed shell;
- the run's parent orch, resolved at call time through `decision add`'s parent resolution;
- the orch strand the named ask went to, so a parent that changed since the ask does not lock out the strand that was asked.

When the run's latest ask is still awaiting an answer, a refusal for an older `--ask` names the latest number.
It says to review the record again under that ask's request.
Any other refusal points to `lym loom decision add <slug> --by parent`, run by the resolved parent orch,
or by the operator's unnamed shell when no named parent resolves.
That covers no ask, an older ask whose latest has ended, an ask already ended, and an ask past its deadline.
The verb writes only the report and the ask's state, never a tracked file or the decision record.

## `lym loom parent-review await [<slug>]`

The chair's wait checks the record once a second, for up to the calling strand's keep-warm cap from `shuttleengines.KeepWarm`.
Once the latest ask has ended, or has passed its deadline (which sets it `timeout`), it answers `ready: true`.
That answer carries the state, a `not-sent` ask's reason and a `done` ask's report path.
When the cap runs out first, it answers `ready: false` and the chair runs it again.
A run with no ask is refused.

## Left to `discussion-review-parent-seat`

This part does not include the steps that need the review round's wiring in `internal/shedadapters`.
Those steps are recording and posting the ask, copying a `done` report beside the accepted review,
and having the review gate close an ask the chair left open.
The record API above is what they build on.
