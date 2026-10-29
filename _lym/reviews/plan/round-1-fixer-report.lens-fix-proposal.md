```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the review proposals the fixer applied as written. The set I checked: F1, F2, F4, F5, F6, F7, F8, F9, F10, F11, F12. F3 is a decision finding and was skipped.

1. **MEDIUM.** Changed text: card 13's closed verdict, "with `Continue` on a dirty tree or an ungated head it keeps waiting, and step 3 gates a moved head first", and its test row that keeps waiting on a dirty tree.
   - Summary: F4's new keep-waiting branch can hang the run with no agent left to clean the tree.
   - Card 14 stops the session "on the continue flag before its re-read". It resumes through `ensureSession` only after an approved read that keeps waiting, or after a pause.
   - So a closed pull request with `Continue` on a dirty tree stops the agent, and nothing restarts it.
   - The tree stays dirty and the wait "has no timeout" (card 13's Bound). The run then waits forever with no halt and no way forward.
   - Before the fix, this branch ended Done. The open-with-`Continue` branch already had this gap, and F4 copies it into the closed branch.
   - Possible fix: card 14 resumes the session when a continue read keeps waiting, as it does for an approved read. Or the closed-and-dirty case halts, naming the uncommitted files.
2. **LOW.** Changed text: card 8's host rule, "an origin is on `github.com` when that host equals `github.com`".
   - Summary: the comparison is case-sensitive, so a mixed-case GitHub origin becomes a silent no under parent `on`.
   - Take `https://GitHub.com/o/r`. Its host does not equal `github.com`, so it gets the not-`github.com` fact. Under parent `on` with an empty field that is `Open: false`, and the run lands locally.
   - Hostnames are case-insensitive, so this is a real GitHub origin. `ParseOwnerRepo` refuses it, since `internal/githubclient/parseownerrepo.go` matches case-sensitive prefixes. That is a refused github.com shape, which D4 says must halt, never give a no.
   - Fix: compare the host case-insensitively, so this origin takes the halt, and add a row for it.
3. **LOW.** Changed text: card 13 step 4, "also at once on the first wake whose fresh record carries `Continue` newly set", against the changed test row "GitHub is polled at most once a minute of the injected clock, also across several wakes of a record carrying `Continue` on a dirty tree".
   - Summary: the one-off immediate poll can come within a minute of the previous poll, which breaks the strict assertion of that row.
   - If the row sets `Continue` mid-wait, the extra poll fails the row's strict once-a-minute assertion.
   - "Newly set" is also undefined when `Continue` is already in the copy `wait` starts with, after a re-entry.
   - Fix: define "newly set" as differing from the previous wake's copy, and state that the row seeds `Continue` before `Call` or allows that one extra poll.

Calls made:
- Read `internal/githubclient/parseownerrepo.go` and `_lym/plan/01-parse-owner-repo-shapes.md`, to check F5's host rule and test rows against the shapes `ParseOwnerRepo` accepts. My context did not hold card 1 or the parser source.
