---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go:289
    summary: A double-quoted gh command, subcommand or method word slips past the publish deny, so gh "pr" merge 3 and gh api -X "POST" are allowed
    fix: In publishCommandPattern let the command word, the subcommand word and the -X or --method value each take an optional JSON-escaped double quote before and after the word (an optional backslash-quote unit), in the alternatives that name pr, review, edit, api and in the write-method alternative, and make notWordPattern's first-character class stop excluding the backslash-quote opener so a quoted unlisted word is denied. Add deny rows to TestRows_PatternsDenyAndAllow for gh "pr" merge 3, gh pr "merge" 3 and gh api -X "POST" repos/o/r/pulls/3/merge. If the quoted spelling is left open on purpose, name it in the publish row's guardrail comment in rows_after_raw_go instead, beside the other passes.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview_wait_test.go:616
    summary: The failed-edit route of syncText has no test row, so the transient retry and the Stuck verdict through pollFailure are unpinned
    fix: Add two TestPRReview_Wait rows beside the new resync rows, each with a rewritten description and a fake API whose Edit fails. One row makes Edit fail transiently once and then succeed, and expects the wait to go on, one Warn, and a second Edit call at the next poll with the same text. The other makes Edit fail with a non-transient error and expects the Stuck verdict with Halt that githubFailure gives.
  - id: F3
    severity: LOW
    class: design
    origin: handler
    location: internal/shellguard/rows.go:296
    summary: Clustered or attached short flags slip past the review and edit alternatives, as in gh pr review 3 -ab x, -rb x and gh pr edit 3 -Bmain
    fix: In the pr review alternative match -a and -r inside a short-flag cluster (a dash, letters, then a or r, as in -[A-Za-z]*[ar]) and in the pr edit alternative match -B followed by any attached value, keeping the word-end rule for the long flags. Add deny rows for gh pr review 3 -rb x and gh pr edit 3 -Bmain. If the cluster form is left open on purpose, name it in the publish row's guardrail comment.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shellguard/rows.go:300
    summary: The mutation alternative stops at a JSON newline, so a GraphQL mutation on a later line of a multi-line quoted query passes
    fix: Let the scan in the gh api mutation alternative cross a JSON-escaped newline while no separator has ended the command, by allowing a backslash-n pair inside the quoted run, and add a deny row for a gh api graphql call whose query puts mutation on a second line. If it is left open on purpose, name multi-line queries beside the mutation-from-a-file hole in the publish row's guardrail comment.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/prreview.go:552
    summary: The resync's open-state and review-ended gates, the zero-baseline branch and the warning re-arm have no test row
    fix: In TestPRReview_Wait add rows, each with a rewritten description and no Edit call expected for the first two. A merged or closed status, and a record carrying ReviewEnded, expect no Edit. A wait entered with a zero synced text and an open status takes the present description as the baseline and edits nothing at the first poll. Extend the no-text row, or add one, with a bad description then a good one then a bad one, expecting two warnings.
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shellguard/rows_integration_test.go:226
    summary: Several publish alternatives and table entries have no deny or allow row, though the change description claims an allow row for every listed form
    fix: Add rows to the publish group of TestRows_PatternsDenyAndAllow. Deny rows for gh pr review 3 --request-changes, gh pr edit 3 -B x, gh api --method=DELETE x, gh api -XPOST x, gh --repo o/r pr merge 3 and gh issue create. Allow rows for gh run watch 5, gh status and gh pr status. Then correct the Testing section of the change description if a form is still unpinned.
  - id: F7
    severity: LOW
    class: scope
    origin: handler
    location: internal/shellguard/rows.go:200
    summary: githubAllowed lets through gh issue, repo view, run list and watch, search, status and pr status, which the entry does not ask for, and the sampler session shares them
    fix: Cut githubAllowed to the entry's set, which is api, pr checks, comment, diff, edit, list, review and view, adding gh run view only if the stencil keeps naming it for a check's log, and update the stencil, doc.go and test rows to match. If the broader read set is wanted, the parent records it with lym loom decision add and this finding is dropped.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/spec.go:49
    summary: Three comments still say the publish switch refuses git push and gh outright, which the allowlist no longer does
    fix: Reword the comment at internal/shuttleengine/spec.go:49, internal/shellguard/rows.go:23 and internal/shuttleengine/claudeengine/settings.go:322 to say git push and the gh calls beyond reads, comments and title and body edits, the wording shellguard/doc.go already uses.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:329
    summary: A persistently failing transient edit warns on every poll because the next good status poll clears the shared flag, and the no-text warning is also logged for a credit or drift read failure
    fix: In syncText keep the transient edit failure's warned flag apart from transientWarned, for example a field cleared only after a successful edit, so a failing edit logs one Warn per episode. Reword the no-text Warn to say no pull request text could be built, since pullRequestText also fails on the credit block and the regression check.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/landingshed/prreview.go:456
    summary: The wait comment says the resync follows a changed description, but a changed regression check or credit block also changes the text and triggers it
    fix: Reword the sentence in the wait doc comment to say the text yielded from the description, the regression check and the credit block changed since they were last set.
---

### [MEDIUM] Quoted gh words slip past the publish deny

**Location:** internal/shellguard/rows.go:289
**Issue:** The generated pattern reads a gh word only as unquoted characters, because the backslash that opens a JSON-escaped quote is in the word class the first character may not take. A command word, subcommand word or method value in double quotes therefore matches no deny alternative. `gh "pr" merge 3`, `gh pr "merge" 3` and `gh api -X "POST" repos/o/r/pulls/3/merge` are all allowed. Before the change every gh in command position was refused, so this is a new hole for exactly the calls the entry keeps with lym: merging, closing and writing methods. Single-quoted forms are still refused. The row's guardrail comment does not list this spelling.

### [MEDIUM] Failed-edit route of the resync is untested

**Location:** internal/landingshed/prreview_wait_test.go:616
**Issue:** `syncText` sends a failed `Edit` through `pollFailure`, and the change description says the edit is judged as a failed poll is. No row makes `Edit` fail. A transient failure that warns and retries with `state.synced` unchanged, and a non-transient failure ending Stuck, can both break with every test green.

### [LOW] Clustered short flags slip past the review and edit alternatives

**Location:** internal/shellguard/rows.go:296
**Issue:** `-a`, `-r` and `-B` match only with a word end after them. `gh pr review 3 -ab x` approves with a body, `-rb x` requests changes, and `gh pr edit 3 -Bmain` changes the base, because pflag accepts clustered shorthand and an attached value. The row is a guardrail, but the comment does not name these spellings.

### [LOW] A mutation on a later query line passes

**Location:** internal/shellguard/rows.go:300
**Issue:** The scan for `mutation` after `gh api` excludes a JSON newline, so a quoted multi-line query that puts `mutation` on a later line is allowed. The comment names only a mutation read from a file.

### [LOW] Resync gates and branches without a test row

**Location:** internal/landingshed/prreview.go:552
**Issue:** Only the continue flag is tested as a gate. Dropping the open-state check or the `ReviewEnded` check, which would edit a merged or closed pull request or an ended review, passes every test. The zero-baseline branch of `syncText` and the reset of `textWarned` are also unpinned.

### [LOW] Publish alternatives and table entries without a deny or allow row

**Location:** internal/shellguard/rows_integration_test.go:226
**Issue:** `-a`, `--request-changes`, `-B`, `PATCH` and `DELETE` methods, `-XPOST` without a space, `--repo=` before the command, and a deny of an unlisted `issue`, `repo` or `run` subcommand have no row, and `gh run watch`, `gh status` and `gh pr status` are never shown allowed. The change description says allow rows exist for every listed form.

### [LOW] Allowlist goes beyond the entry's wanted set

**Location:** internal/shellguard/rows.go:200
**Issue:** The entry names `gh pr view`, `list`, `diff` and `checks`, `gh api` reads, `gh pr edit` for title and body, `gh pr comment` and `gh pr review` comments. The table adds `gh issue list` and `view`, `gh repo view`, `gh run list`, `view` and `watch`, `gh search`, `gh status` and `gh pr status`. All are reads, but the sampler session shares the switch, so it also gains `gh pr edit` and `gh pr comment` on a pull request, which the entry did not ask for. The change description records the broadening as a design decision, and no `Added after Discussion` entry licenses it.

### [LOW] Stale comments describe the switch as refusing all of gh

**Location:** internal/shuttleengine/spec.go:49
**Issue:** The comments at `internal/shuttleengine/spec.go:49`, `internal/shellguard/rows.go:23` and `internal/shuttleengine/claudeengine/settings.go:322` still say the publish deny refuses git push and gh. The row now refuses git push and only the gh calls outside `githubAllowed`.

### [LOW] Edit failure warns every poll; no-text warning misnames its cause

**Location:** internal/landingshed/prreview.go:329
**Issue:** A transient `Edit` failure sets `state.transientWarned`, and the next good `pollStatus` clears it before the next sync, so a persistent failure logs one Warn per poll instead of one per episode. The no-text Warn is also logged when the credit block or the regression check fails to read, since `pullRequestText` returns all three errors alike.

### [NIT] The wait comment names only the description as the trigger

**Location:** internal/landingshed/prreview.go:456
**Issue:** The text includes the regression check and credit block, so a change in either also resyncs the body. The comment says the resync follows a change of the description.

## Rejected

- Whether the stencil's "your settings deny them too" holds: the Claude and Codex settings carry no gh deny besides the shell guard row, which they install as a hook, so the sentence holds through the hook.
- The `gh api` implied POST with field flags, a raw tab between words and `env X=1 gh`: the entry's own guardrail comment accepts them, or the row already lists them as passes.
- The failed-edit route ending the run Stuck on a permanent GitHub refusal: it is the same route a failed poll takes, which the change description states, and the Stuck reason names the way forward.
