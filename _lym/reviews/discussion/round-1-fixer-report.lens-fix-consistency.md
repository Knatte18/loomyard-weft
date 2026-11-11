```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling decisions, criteria, cross-references and support-log entries it touches.

Findings, three in total, all LOW:

1. **LOW: the D3 rationale still names the old fallback rule.**
   - **Changed text:** D3 step 4 and its criterion, which now say "the candidate whose latest `resets_at` among its blocking windows is earliest".
   - **Summary:** step 4 was reworded, but the D3 rationale was left as it was.
   - **Evidence:** the rationale still says "Falling back to the soonest reset keeps work starting when every account is near a limit (D8)". That is the reading F9 called ambiguous.

2. **LOW: a rejected alternative in the support log now reads against D5.**
   - **Changed text:** D5's opening, "When the worktree's recorded directory is a listed account's, compared cleaned (D1)… The engine resolves that account at launch", and its new rationale sentence.
   - **Summary:** D5 now picks the account by matching directories at launch, but the support log still rejects matching by path, on grounds that are no longer accurate.
   - **Evidence:** the support log's Rejected alternatives still says "Matching the status line's `CLAUDE_CONFIG_DIR` to an account by path: a differently spelled path records nothing; the account name passed as an argument cannot miss." The rejected thing was path matching inside the verb, and the new design still passes a name to the verb. But "cannot miss" no longer holds: a session started before a rename now passes a name the verb refuses.

3. **LOW: support-log Q7 states the old precedence rule.**
   - **Changed text:** D2's new first bullet, "With `claude.accounts` empty … A record already there is still read and its directory exported".
   - **Summary:** under the new D2, a record's directory still wins over an `env` value when the list is empty.
   - **Evidence:** Q7 in the question ledger still answers "The account wins when accounts are configured; otherwise the `env` value stands."

Tool calls: none. My context already held both changed files and the review.
