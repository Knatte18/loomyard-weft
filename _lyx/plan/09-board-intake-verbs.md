# Card 9 — board-intake-verbs

**Edit:**
- `internal/boardcli/cli.go`

**Create:**
- `internal/boardcli/intake.go`
- `internal/boardcli/intake_test.go`

**Uses:**
- `internal/boardengine/intake.go`
- `internal/selfreportengine/inbox.go`

**Intent:**
D10: the `lyx board intake` verb group, JSON in and out, against the inbox repo only; it does the mechanics and never triages.
`boardcli` wires `boardengine` (the store side) and `selfreportengine` (the GitHub side), as D11 decides.

`intake.go` holds the group constructor and one handler per verb, each taking the `Board` and the payload so tests can call it without resolving a cwd:
- `list`: `ListOpenIssues`, minus every issue `Board.RecordedIssues` names, printed as `{"issues":[…]}` with number, title, body, labels, URL and creation time.
- `import '{"issue":N,"slug":…,"title"?,"brief"?,"labels"?}'` or `import '{"issue":N,"into":"<slug>"}'`: unknown keys are refused;
  `GetIssue` first, then `Board.ImportIssue`.
  A no-op prints the recording entries and touches nothing on GitHub.
  Otherwise the issue gets the comment ``Imported into the board as `<slug>`.`` and is closed as completed through `CommentAndClose`, after the lock is released.
  When that GitHub step fails, the failure envelope (`output.ErrFields`) carries the written entry and the exact `close` payload to run: `{"issue":N,"reason":"Imported into the board as `<slug>`.","completed":true}`.
  The success result carries the entry and the dropped labels.
- `close '{"issue":N,"reason":"…","completed"?:true}'`: `reason` is required and non-empty;
  `GetIssue`, then a pull request is refused and an already closed issue is refused naming its state and state reason;
  otherwise `CommentAndClose` with the reason, as not planned or, with `completed`, as completed.
  It writes nothing to the board and never changes the issue's content; it is also the way to close an issue left open after a failed import close, recorded or not.
  Bound: it closes any open issue of the inbox repo the caller names, because closing noise is its purpose; it never deletes or edits an issue, and never touches another repo.

`cli.go` adds the group under `board`.
The `PersistentPreRunE` skips config and cwd resolution for the bare `intake` group, as it does for `board` itself, so a bare or bogus `lyx board intake` lists or refuses without the "not initialized here" envelope and the clitree walk passes with no new `cliTreeFindings` entry.
Every verb has a `Short`, help naming its payload keys, and JSON errors through `internal/output`; `clihelp.WrapRun` supplies the `ShouldAbort` check (PATTERN-cli-cobra).

Tests (`intake_test.go`, untagged: an `httptest` server through `selfreportengine.NewGitHubClient` and a `Board` over a temp dir with `SkipGit`):
`list` excluding pull requests, recorded issues and closed issues across two pages;
`import` creating a note then commenting and closing as completed, folding with `into`, and a recorded issue sending no GitHub mutation;
an import refused before any board write or GitHub mutation (no usable type label, a taken slug, a missing `into`), asserting the server saw no comment or close and `board.json` is unchanged;
a close failure after the write reporting the entry and the `close` payload;
`close` without a reason, on a pull request, on a closed issue, with and without `completed`, and on a recorded open issue, each asserting `board.json` is unchanged.

**ImpactSummary:** Adds one verb group under `lyx board` and one early return in its pre-run; no existing verb changes.
