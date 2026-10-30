# Card 7 — pullrequest-record-and-client

**Create:**
- `internal/pullrequest/doc.go`
- `internal/pullrequest/testmain_test.go`
- `internal/pullrequest/record_test.go`
- `internal/pullrequest/client_test.go`
- `internal/pullrequest#Record`
- `internal/pullrequest#ReadRecord`
- `internal/pullrequest#WriteRecord`
- `internal/pullrequest#UpdateRecord`
- `internal/pullrequest#RemoveRecord`
- `internal/pullrequest#ErrNoRecord`
- `internal/pullrequest#State`
- `internal/pullrequest#Status`
- `internal/pullrequest#Ref`
- `internal/pullrequest#API`
- `internal/pullrequest#Client`
- `internal/pullrequest#Open`
- `internal/pullrequest#NewWithClient`
- `internal/pullrequest#Client.FindOpen`
- `internal/pullrequest#Client.Create`
- `internal/pullrequest#Client.Edit`
- `internal/pullrequest#Client.Status`
- `internal/pullrequest#Client.Close`
- `internal/pullrequest#Client.Merge`
- `internal/pullrequest#standingReviews`
- `internal/pullrequest#MergeRefusal`
- `internal/pullrequest#RefusalKind`
- `internal/pullrequest#TestRecord_RoundTripAndUpdate`
- `internal/pullrequest_test#TestClient_Calls`
- `internal/pullrequest#TestStandingReviews`
- `internal/shedrun#PullRequestRecord`
- `internal/shedrun#PullRequestRecordLock`

**Edit:**
- `internal/shedrun#TestPathConstructors`

**Extends:**
- `docs/overview.md`

**Uses:**
- `internal/githubclient#ParseOwnerRepo`
- `internal/githubclient#NewWithBaseURL`
- `internal/shedrun#ScratchDir`
- `internal/shedrun/doc.go`

**Intent:**
D6, D9, D11 and D12 share one record of a run's pull request and one set of GitHub calls, so they live in a new package `internal/pullrequest`, built like `internal/hubissues`: it authenticates only through `githubclient` and calls go-github directly, and runs no `gh`.

The record: `Record` holds `Number int`, `URL`, `Base`, `Head` (the task branch), `GatedHead` (the head the PR-Review row last gated and pushed), `MailedHead` (the head whose gate failure was mailed), `StrandGUID`, `StrandName`, `SessionID`, `Engine` (the D7 session), `Continue bool` (the operator's continue flag, set by `lym loom pr-continue`) and `ClosedForContinue bool` (the row closed the pull request on that flag), with snake_case JSON tags and `omitempty` on every field but `number`.
`ReadRecord`, `WriteRecord`, `UpdateRecord` and `RemoveRecord` go through `internal/state` (`ReadJSON`, `WriteJSON`, `UpdateJSON`), so an unknown key survives a rewrite (`PATTERN-tolerant-records`); an absent file reads not found, `UpdateRecord` on an absent record returns `ErrNoRecord` and writes nothing, and `RemoveRecord` of an absent file is no error.
`shedrun.PullRequestRecord` is `pull-request.json` under `ScratchDir(l, runID)` and `PullRequestRecordLock` is `pull-request.json.lock` beside it, so the record is never tracked and goes with the pair.

The calls: `Client` is bound to one repository; `Open(originURL)` parses the URL with `githubclient.ParseOwnerRepo` and builds the go-github client with `githubclient.New`, and `NewWithClient` binds a told client, which tests point at an `httptest` server through `githubclient.NewWithBaseURL`.
`API` is the interface of the six methods below, which landingshed and pairteardown hold so their tests fake it.
`FindOpen` lists open pull requests filtered by `<owner>:<head>` and base and returns the first; `Create` opens one with title and body; `Edit` sets title and body; `Close` posts one issue comment, then sets the state closed.
`Status` reads the pull request and its reviews: `State` is `StateOpen`, `StateClosed` or `StateMerged` (merged wins over closed), `HeadSHA` is the pull request's head SHA, and `Approved` and `ChangesRequested` come from `standingReviews` over every review page.
`standingReviews` keeps each reviewer's latest `APPROVED`, `CHANGES_REQUESTED` or `DISMISSED` review, ignoring `COMMENTED` and `PENDING`; `approved` is true when at least one standing review is `APPROVED` with a commit id equal to head, and `changesRequested` when any standing review is `CHANGES_REQUESTED`.
`Merge` calls the merge endpoint with the expected head SHA, `squash` or `merge` as the method, and the title and message as the commit title and message.
A refused merge (HTTP 405 or 409) returns a `*MergeRefusal` holding GitHub's message and a `RefusalKind`: `RefusalMethod` when the message says the merge method is not allowed, `RefusalPending` when it says the base or head was modified or mergeability is still being computed, and `RefusalOther` for everything else (a moved head, a missing approval, a required check, a conflict); any other failure is returned wrapped, so `githubclient.IsTransient` still judges it.
Every call takes the caller's context; the client's own 30-second bound applies per request.

Tests: `internal/pullrequest/record_test.go`, `//go:build tier0`, holds `TestRecord_RoundTripAndUpdate` (write, read, an unknown key kept across `UpdateRecord`, `UpdateRecord` on an absent record returning `ErrNoRecord`, removal idempotent) and `TestStandingReviews` (rows: a later `COMMENTED` after `APPROVED` keeps the approval; an approval of an older head is no approval; `DISMISSED` after `CHANGES_REQUESTED` clears it; one reviewer's change request beside another's approval reads both).
`internal/pullrequest/client_test.go`, `//go:build tier0`, package `pullrequest_test`, holds `TestClient_Calls` against one scripted `httptest` server: `FindOpen` sends the head and base filters and reads none and one; `Create`, `Edit` and `Close` send the expected method, path and body, `Close` commenting before closing; `Status` maps merged, closed and open with reviews across two pages; `Merge` sends `sha`, `merge_method` and the commit title and message, and maps a 405 method message, a 405 base-modified message and a 409 head-moved message to the three kinds; `Open` refuses a non-GitHub origin.
`internal/pullrequest/testmain_test.go` is untagged support: it runs the package under `tmuxkit.Main`, as every package does.
`TestPathConstructors` gains the two paths.
`doc.go` records the package: the record, its fields, the calls and the refusal kinds, and that it files no issue.
`docs/overview.md`'s module table gains an `internal/pullrequest/` line beside `internal/hubissues/`: the run's pull-request record and its GitHub calls, for PR-Review, Finalize and pair teardown.

**ImpactSummary:** A new package and two new shedrun paths; `TestPathConstructors` is the only existing code touched.

**Commit:** 7: pullrequest: the run's pull-request record and its GitHub calls
