```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Cases run per function or branch (`publishCommandPattern`, `notWordFrom`, `syncText`, `wake`, `wait`):**

- `notWordFrom` / `notWordPattern`, run on `gh pr merge`, `gh pr view`, `gh prs`, `gh api`, `gh ap`, `gh --version`: correct. A word equal to a listed word is not matched, and any prefix or extension of it is. A leading dash never matches, and the `?` on intermediate groups gives the right result for a bare prefix such as `p` or `ap`.
- `publishCommandPattern`, git alt (`git -C x push`): correct.
- `publishCommandPattern`, alt 2 (`gh release create`, `gh auth token`, `gh prs`, `gh -R o/r release`): correct.
- `publishCommandPattern`, per-command subcommand alts (`gh pr merge`, `close`, `create`, `update-branch`, `gh pr -R o/r merge`, `gh --repo o/r pr merge`, `cd x && gh pr merge`): correct, all refused. `gh pr view`, `gh pr list` and `gh pr view | head` pass.
- `publishCommandPattern`, review/edit alts: `--approve`, `-a`, `-r`, `--request-changes`, `--base`, `-B`, `--base=main` are refused. **Incorrect for** `-rb x` (see F2).
- `publishCommandPattern`, `gh api` alts: `-X POST`, `-XPOST`, `--method=put`, `--input`, and `mutation` in the same line are refused. GET and `-f` queries pass. **Incorrect for** quoted methods and newline-split queries (see F1 and F3).
- `steerPublishDeny`, `joinWords`, `checkTexts`: correct. The steer has no forbidden characters; `joinWords` handles one word and several.
- `pullRequestText`, entry path: correct. Parse, credit and drift errors are wrapped as `landingshed: <name>: ...`, with the same text as before the refactor.
- `openPullRequest`, found/open path: `text` reaches `wait` after the `Edit`, so the first sync sees an equal text and edits nothing. Correct.
- `openPullRequest`, `ReviewEnded` and non-open paths (`prText{}`): correct. The `ReviewEnded` path never syncs; a non-open PR cannot sync because the `State == Open` guard in `wake` blocks it. The zero-`synced` branch in `syncText` is therefore close to unreachable, and harmless.
- `syncText`, parse failure: logged once per episode, resets on success, edits nothing. Correct. Wording is wrong when the failure comes from credit or drift (see F4).
- `syncText`, `Edit` success: sets `synced`, so there is no double edit on the next poll. Correct.
- `syncText`, `Edit` non-transient or token failure: ends as Stuck through `pollFailure`, same route as a failed poll. Correct.
- `syncText`, `Edit` transient failure: `synced` is unchanged, so it retries at the next poll. The Warn dedup is wrong (see F4).
- `wake`, guard `open && !ReviewEnded && !Continue`, run after `polled` and before the stop and judge steps: correct. A merged, closed, continued or ended review never edits.
- `wait`, `synced` hand-off into `prWaitState`: correct.

**Findings**

- **F1 · MEDIUM · design** — `internal/shellguard/rows.go:289-307` (`publishCommandPattern`, `ghWordClass` at 237). A double-quoted word is not read as a gh argument word. `gh "pr" merge 3` and `gh pr "merge" 3` match no deny alt, because `\` is in `ghWordClass` and the first character of a word may not be `\`. `gh api -X "POST" repos/o/r/pulls/3/merge` also passes, because the method alt expects `P` straight after `-X `. Before this change every `gh` in command position was refused, so these spellings are a new hole for exactly the calls the entry keeps with lym (merge, close, write methods). Single-quoted forms are still refused, since `'` is not in `ghWordClass`. The row's guardrail comment (506-509) does not list this hole. Fix: let the command and subcommand position, and the method value, accept an optionally quoted word: `\\"?` before and after, or `'`/`\"` quote units. Or at least add the quoted spelling to the comment's list of passes, plus a test row.
- **F2 · LOW · design** — `internal/shellguard/rows.go:296` (`gh pr review` alt). The flags `-r` and `-a` are matched only when a non-word character follows. A combined short-flag cluster such as `gh pr review 3 -rb "x"` or `-ab` passes, though it requests changes or approves. Fix: allow the cluster form, e.g. `-[a-zA-Z]*[ar][a-zA-Z]*`. Or name it in the guardrail comment.
- **F3 · LOW · design** — `internal/shellguard/rows.go:300` (`mutation` alt). `([^;&|\\]|\\[^n])*` stops at a JSON `\n`, so a GraphQL mutation on a later line of a multi-line quoted query passes: `gh api graphql -f query='` newline `mutation {...}'`. The comment (507) names only a mutation read from a file. Fix: let the `mutation` scan cross `\n` while inside the same `gh api` command, or name multi-line queries in the comment.
- **F4 · LOW · consistency** — `internal/landingshed/prreview.go:329-351` (`syncText`). Two small defects:
  - The Warn "the change description yields no pull request text" is also logged for a credit-block or regression-check read failure, because `pullRequestText` returns all three errors alike. The `cause` field carries the real error, but the message misleads. Fix: word the Warn as "no pull request text could be built".
  - A transient `Edit` failure sets the shared `state.transientWarned`, but the next successful `pollStatus` (`prreview.go:693`) clears it before the next sync. A persistently failing edit therefore logs one Warn per poll rather than one per episode, contrary to what `pollFailure` documents for its transient failures. Fix: give the edit its own warned flag, or leave `transientWarned` set when the poll succeeds and only the sync failed.
- **F5 · LOW · consistency** — `internal/landingshed/prreview.go:308-322` against the `wait` comment (456) and `doc.go`. `pullRequestText` includes the regression-check section (`driftVerdict`, `finalize_pr.go:82`) and the credit block. A change in the drift verdict alone, with the description untouched, therefore also triggers a resync of the body. That is arguably desirable, but the comments say the resync happens only "when the change description changed". Fix: reword the comments to "when the text yielded from the description, the regression check and the credit block changed".

Accepted per the entry or the existing guardrail comments, not findings: the implied-POST hole for `gh api` with `-f` fields, a raw tab (`\t`) between words, and `env X=1 gh ...`.

**Tool calls**
1. `grep` in `internal/landingshed/*.go` for the `driftVerdict` and `creditBlock` definitions. My context did not have their bodies, and I needed to see whether they can change between polls and what their errors say.
2. `grep` of `githubclient.go` for `Edit(ctx`. I wanted to confirm what the edit sends; it returned nothing, so the finding set is unaffected.
