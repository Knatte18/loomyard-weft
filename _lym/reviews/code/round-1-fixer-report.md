# Fixer report, code review round 1

All ten findings are fixed.
`lym gate test` passed at tier 0 for `internal/landingshed`, `internal/shuttleengine` and `internal/shuttleengine/claudeengine`, and at tier 1 for `internal/shellguard`.
The self-check fixes were gated again for `internal/landingshed` at tier 0 and `internal/shellguard` at tier 1, and passed.

## Changes

Commits are grouped by file, so one commit can carry several findings.

- `7bf7afe3` shellguard: F1, F3, F4, F6, F7.
- `77572456` shuttleengine: F8.
- `ce42380b` landingshed: F2, F5, F9, F10.
- The next commit after `ce42380b`, the self-check fixes: the `PRReview` type doc, `editWarned` cleared on an unchanged text, and the apostrophe pass named in the publish row's comment.

### F1 — fixed differently
The proposed change was to stop `notWordPattern`'s first-character class from excluding the backslash.
That would deny `gh "pr" view`, because a quoted listed word would then read as an unlisted one.
Instead `notWordPattern` takes an optional JSON-escaped quote before the word, and the first-character class still excludes the backslash.
A quoted listed word fails the not-word match, because its closing quote is outside the word class, and a quoted unlisted word matches it.
The command word, the subcommand word and the `-X` or `--method` value take optional quotes on both sides in the `pr`, `review`, `edit` and `api` alternatives.
Deny rows: `gh "pr" merge 3`, `gh pr "merge" 3`, `gh api -X "POST" ...`, `gh "release" create v1`, `gh pr "review" 3 --approve`.
Allow rows: `gh "pr" view 3`, `gh pr "view" 3`, `gh api -X "GET" ...`.

### F2
Three `TestPRReview_Wait` rows: a transient edit failure that retries the same text at the next poll with one Warn, a persistent transient failure retried at every poll with one Warn, and a non-transient failure that ends Stuck with Halt.

### F3
The `pr review` alternative matches `-a` and `-r` anywhere in a short-flag cluster, and `pr edit` matches `-B` with any attached value.
Deny rows: `-ab x`, `-rb x`, `--request-changes`, `-B x`, `-Bmain`.
A cluster whose attached body holds an `a` or an `r` is falsely denied, and the publish row's comment says so.
The allow row `-cb "a nit"` keeps a comment cluster free.

### F4
The `gh api` mutation scan now reads quoted strings whole, so it crosses a JSON newline inside a quoted string only, and stops at an unquoted separator or newline.
It also stops at the JSON closing quote, so it no longer reaches the `description` field.
Deny rows: a mutation on a later line of a single-quoted query, and a mutation in a double-quoted query.
Allow row: `gh api ...` followed by a `grep mutation` command on the next line.

### F5 — fixed, in part by a different change
Rows added: an edited description is not set on a pull request closed at the poll, nor on one merged at the poll; a bad, good, bad description logs two warnings.
A separate test calls `wait` with no last-set text and an open status, and expects no `Edit` at the first poll.
No row pins the `ReviewEnded` gate in `wake`.
A record with `ReviewEnded` on an open pull request ends the call at the first wake, through `judge`, before any description change is polled, so no assertion can tell the gate from its absence.
The existing row `ReviewEndedOnAnOpenPullRequestOnACleanTreeIsDoneWithoutEditingIt` already shows no `Edit` on that path.

### F6 and F7 — read together
F7 cuts `githubAllowed` to `api`, `pr` (checks, comment, diff, edit, list, review, view) and `run view`, which the stencil still names.
F6 asked for allow rows for `gh run watch`, `gh status` and `gh pr status`, but F7 makes those forms denied, so those rows are deny rows.
The row `gh_search` flipped from an allow to a deny row.
Deny rows also added: `gh issue create`, `gh issue list`, `gh repo view`, `gh run list`, `gh --repo o/r pr merge 3`, `gh --repo=o/r pr merge 3`, `gh api --method=DELETE x`, `gh api -XPOST x`.
The stencil and `doc.go` needed no change: the stencil names only forms the cut set keeps, and `doc.go` describes the table by its kind of calls.
The generated steer follows the table.

### F8
The three comments (`internal/shuttleengine/spec.go`, `internal/shellguard/rows.go`, `internal/shuttleengine/claudeengine/settings.go`) use `doc.go`'s wording.

### F9
`pollFailure` takes a pointer to the warned flag.
The status poll passes `transientWarned`, and the edit passes a new `editWarned`, cleared by an edit that succeeds or by a text that matches the last-set one.
The no-text Warn reads "no pull request text could be built".
The persistent-failure row in F2 fails under the old shared flag.

### F10
The `wait` doc comment names the description, the regression check and the credit block as the text whose change resyncs the pull request.

## Self-check

- fix-consistency, LOW: the `PRReview` type doc still named the description alone as the trigger — fixed as proposed.
- fix-consistency, LOW: the change description still records the broadened gh set and claims an allow row for every listed form — deferred, see below.
- fix-proposal, LOW: an unbalanced apostrophe before the mutation word passes the quote-aware scan — left open on purpose and named in the publish row's comment, since the input is not valid shell.
- fix-code-correctness, LOW: `editWarned` stayed set when the text went back to the last-set one — fixed as proposed.
- fix-code-correctness, LOW: no row fails if the `ReviewEnded` gate is dropped — not fixed; the gate cannot be pinned, see F5.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- The change description's design note on the broadened read set and its Testing section no longer match the table and the rows after F7.
  The description is not in this fixer's write surface; the parent or the operator updates it, or records the broader set with `lym loom decision add` and drops F7.
