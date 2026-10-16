# Card 1 — intake list prints numbers and titles unless --full

**Create:**
- `internal/boardcli#intakeIssueSummary`

**Edit:**
- `internal/boardcli#intakeCommand`
- `internal/boardcli#intakeList` -> `func intakeList(out io.Writer, b *boardengine.Board, full bool) int`
- `internal/boardcli#intakeIssueView`
- `internal/boardcli#fakeInbox`
- `internal/boardcli#fakeInbox.list`
- `internal/boardcli#TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages`
- `internal/boardcli/doc.go`

**Uses:**
- `internal/clihelp#RunRootCtx`
- `internal/boardcli#newFakeInbox`
- `internal/boardcli#newIntakeBoard`
- `internal/boardcli#runHandler`

**Intent:**
D1: `lyx board intake list` prints numbers and titles by default, and the full view moves behind `--full`.

- `intakeIssueSummary` is the compact view, with `Number int` tagged `json:"number"` and `Title string` tagged `json:"title"` and no other field.
- `intakeList` takes `full`.
  Filtering stays as it is in both modes: pull requests are already left out by `selfreportengine.ListOpenIssues`, and an issue `b.RecordedIssues` names is skipped.
  With `full` false it appends an `intakeIssueSummary` per kept issue; with `full` true it appends today's `intakeIssueView`.
  Each mode starts from an empty, non-nil slice, so an empty inbox prints `{"ok":true,"issues":[]}` in both.
  A `ListOpenIssues` or `RecordedIssues` error prints the usual error envelope through `outputError` in both modes, unchanged.
- `intakeCommand`'s `list` command gets a `--full` bool flag (`cmd.Flags().BoolVar`, as `prune`'s `--apply` in `cli.go` does), and its `RunE` passes the flag to `intakeList`.
  Its `Long` says the default prints `{"issues":[...]}` with number and title for each issue, `--full` adds body, labels, url, comments (the comment count) and created_at, and the filtering is the same in both modes; the example adds `lyx board intake list --full`.
  `--full` is read-only and prints the same filtered set: it lets through only the bodies, labels and links the default leaves out, and reaches no other issue and no write.
- `intakeIssueView`'s doc comment says it is the view `list --full` prints.
- `fakeInbox` gets a `failList bool` field, commented like `failClose`, and `fakeInbox.list` answers a server error (`http.Error` with `{"message":"boom"}` and status 500) when it is set.
- `TestIntakeList_ExcludesPullRequestsRecordedAndClosedAcrossPages` becomes table-driven, each row carrying its own `fakeIssue` list, `failList` setting, command arguments and expected outcome.
  Every row seeds the board entry `kept` recording issue 4, as today, and runs the `list` command that `intakeCommand(func() *boardengine.Board { return b })` builds, with the row's arguments set through `SetArgs` and its output through `SetOut`, under `clihelp.RunRootCtx`, so a row reaches `intakeList` through the cobra flag and not through a handler argument.
  Each row keeps the no-per-issue-request assertion.
  - compact, arguments `list`, over today's inbox (issues 1 to 6 across pages, the pull request 2, the closed 3, the recorded 4): numbers `[1 5 6]`, and every issue object has exactly the keys `number` and `title`, the first with title `Issue 1`.
  - full, arguments `list --full`, over the same inbox: numbers `[1 5 6]`, today's field assertions on the first issue (title, body, url, created_at), the comment counts 2 and 0, and issue 6's `labels` equal to `[bug]`.
  - empty inbox, once per mode: an inbox of only the closed issue 3, the pull request 2 and the recorded issue 4 gives exit 0 and an `issues` value that is an empty JSON array, not null or absent.
  - list failure, arguments `list`: `failList` set gives exit 1, `ok` false, a non-empty `error` and no `issues` key.
    One row suffices, since the fetch fails before the mode is read.
  - board load failure, arguments `list`, over today's inbox: the row writes `{` to the board.json path `newIntakeBoard` returns, after the seed, so `b.RecordedIssues` fails, and asserts exit 1, `ok` false, a non-empty `error` and no `issues` key.
- `doc.go` gains a paragraph stating that `intake list` prints `{"issues":[{"number":…,"title":…}]}` for each open inbox issue no entry records, pull requests excluded, an empty inbox as `{"issues":[]}`, and that `--full` prints number, title, body, labels, url, comments and created_at for the same set.

**ImpactSummary:** `intakeList` has one production caller, `intakeCommand`'s `list` RunE, which its one test now drives; no other package reads the list output.

**Commit:** 1: intake list prints numbers and titles unless --full
