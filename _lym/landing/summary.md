# Add the parent-review ask record, deadline keys, answer and await verbs

A Discussion-Review round can now ask the run's parent orch to review the decision record beside the reviewer.
This change lays the groundwork: two config keys, the ask record, the capped deadline, and the two verbs the parent and the chair use.
Nothing posts an ask yet; the review row that does is a later change.

## What changes

- `loom.yaml` gains `discussion_review_ask_parent` (default `false`) and `discussion_review_parent_timeout_min` (default `30`).
  Both are open to `run_settings`, and the template comments say so.
  Config validation refuses a timeout below 1, naming the key.
- `internal/burlerengine` gains the ask record and the parent report, both under the worktree's `.lym/parent-review` and never tracked.
  The record is Go-written JSON through `internal/state`.
  The API reads and writes the record, and moves an ask's state forward only:
  `pending-post` to `open`, and either to `done`, `timeout`, `no-parent` or `not-sent`.
  Every writer takes one per-run lock, so an answer and a timeout never both land.
- `internal/loomengine` gains `CapParentReviewWait`, which follows `CapAssistTimeout`.
  The wait is the lesser of the configured minutes and the chair's timeout less shuttle's start allowance,
  one keep-warm await cap, and `ParentConsolidationMargin` (5 minutes).
  A capped wait logs one Warn naming both keys.
  A room of zero or less returns a zero wait and a reason; the round then sends no ask and records `not-sent`.
  `chairTimeout` is factored out of `CapAssistTimeout` and the two share it.
- `lym loom parent-review answer <slug> --ask <n>` takes exactly one of `--satisfied` or `--findings-file <path>`.
  It writes the report with ids `P1`, `P2` and so on in file order, and sets the ask `done`.
  The findings file is a YAML `findings` list; each item needs a `change` and may carry a `severity` and `class` from the review format's words.
- `lym loom parent-review await <slug>` answers `ready: true` once the ask is `done`, `timeout`, `no-parent` or `not-sent`.
  At or past the deadline with the ask still open it sets `timeout` and answers ready.
  Otherwise it wakes at most once a second up to the chair's keep-warm cap, then answers `ready: false`.
  A ready answer carries the state, the `not-sent` reason, and the report path for `done`.
- `docs/overview.md` lists the verbs and keys.

## Design decisions

- The answer guard is the one `lym loom decision add --by parent` uses: the run's parent orch and the operator's unnamed shell.
  It also admits the strand the ask went to, so an orch that came up or went down between ask and answer is not locked out.
  The guard checks an unverified strand name, so it stops a mistaken call, not a forged one.
- The answer accepts an ask in `pending-post` or `open`, since mail may have gone out before a crash left the state unset.
  An ask at or past its deadline is set `timeout` and the answer refused.
- Every refusal names its way forward: a call the parent still wants goes in with `lym loom decision add <slug> --by parent`,
  and the refusal names who runs it.
  An `--ask` older than a live latest ask is told to review the record again and answer the latest number.
- The await is open to any caller and writes only `timeout`, only to an ask at or past its deadline.
- The capped deadline is the longest the parent can hold a round.

## To check by hand

- The decision record also calls for closing an unanswered ask when the chair's review is accepted, and for copying a `done` report beside the review file.
  Neither is in this diff; confirm the review row that posts asks takes both on.
- No live `lym loom parent-review answer` or `await` run against a real creel ask was made; the tests cover the verbs and the record.
