```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches. I found no findings.

**Per change, the set found and whether it is covered**

1. **`pr-continue` now clears `MailedHead`; its idempotent branch is `!ReviewEnded && (!Continue || MailedHead != "")`.**
   - Writers and readers of `MailedHead` in non-test code are only `gateDue` (`prreview.go:622`), `gateHead` (`:747`, `:751`), `prContinueVerb`, and the `Record` field (`record.go:28`). All are consistent with the change. Covered.
   - Docs that describe the verb are updated: `prcontinue.go` header, `prContinueVerb` doc, the cobra `Long` text, `loomcli/doc.go:577`, `landingshed/doc.go:36`, `docs/overview.md:1304`, and the `record.go:26-27` field comment. The `record.go:34` Continue comment ("set by `lym loom pr-continue`") is still true. Covered.
   - A repo-wide grep for `pr-continue` and `MailedHead` found no other doc, stencil, fixture or PATTERN entry that describes the old "sets the flag and nothing else" behaviour. `pattern/` and `PATTERN.md` mention neither term. Covered.

2. **`sessionStopWanted` gets a new signature, `(status, rec, head, dirty)`.**
   - Its one non-test caller is `prreview.go:653`, which is updated. The grep found no other caller, and none in test files either. The old `docs/research/turns/...` hits are historical measurement data, not subjects. Covered.
   - `landingshed/doc.go` (lines ~73-79) is updated to the new stop conditions. Covered.
   - `docs/overview.md` around lines 1301-1307 says nothing about the session stop condition, so nothing is left stale. Covered.

3. **The mail text gains `gateRetryHint`.**
   - The mail is asserted in `prreview_wait_test.go` around line 457, and that assertion now includes `"lym loom pr-continue"`. Covered.
   - `landingshed/doc.go:35-37` and `docs/overview.md:1304` describe the mail. Covered.
   - The stencil's new step 3 matches the hint's condition: a failure outside the change, after the operator asked to continue. Covered.

4. **The PR-Review stencil gets a new step 3.**
   - This is the stencil's only copy in the repo (embedded default, `landing-template-pr-review.md:62-67`). It drops or weakens no existing rule, and it is within the board entry's Expected section.
   - Its markers are unchanged, and it adds no `{{if}}` and no new substitution. PATTERN-stencil-ownership is not touched. Covered.

5. **Repository rules.**
   - No Pattern-Departure trailer is needed. The commit messages carry none, and the diff touches no PATTERN-governed boundary.
   - No refusal message was added or changed, so PATTERN-refusal-way-forward is not touched.
   - There are no declared PATTERN changes to land.

Tool calls:
1. Grep for `pr-continue`, `once per head` and `gate failure` across docs, contracts and internal. I needed the full set of docs and stencils that describe the changed behaviour, and my context only held the diff.
2. Read the stencil's first 80 lines, listed `pattern/`, and grepped it and `PATTERN.md`. I needed the stencil's marker header and the PATTERN coverage of the changed terms.
