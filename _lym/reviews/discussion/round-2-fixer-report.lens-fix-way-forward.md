```yaml
lens: fix-way-forward
outcome: reported
```

Checked the ways forward the round-2 fix introduced in D5, D6, D7 and the D5 criterion.

**[MEDIUM] A refused code-side push now restarts on every dobby tick.**
- **Changed text:** D5's dobby-tick bullet ("a pair is ahead when its local records branch or its local code branch is") and the matching D5 criterion ("a pair whose records branch is level and whose code branch is ahead included").
- **Summary:** the tick can now start a pusher over and over without making progress.
- **Evidence:** D3's start condition counts only "no refused-push record stands for its records branch (D5)". When a pair's code-side push is refused, the pusher leaves its record against the code branch and exits. The records branch now reads level and the code branch stays ahead. Before this fix, the tick read only the records branch and started nothing, so only a new commit restarted the pusher. Now every dobby tick passes the start conditions, starts a pusher, meets the refusal again and posts another notice. This happens even for a halted run that makes no commits. D5's own claim that the dobby does not start the pusher again "while a refused-push record stands for the worktree's branch" does not hold, because the gate never reads the code branch's record. The review's Rejected list already names this loop at commit frequency. The fix moves it to tick frequency.
- **Way forward:** have D3's refused-push start condition read the records of both of a pair's branches, records and code.

I checked the other ways forward and they hold:
- D7's "until `lym tender resume` ends the entry" can be carried out. The pre-push notice names `lym tender resume &lt;slug&gt;` (`internal/loomcli/prepush.go:223`), and once the entry is gone the next dobby tick finds the pair ahead and starts the pusher.
- D6's sentence moving the restart of a held tip to D5's tick holds once the code-branch case is in D5.

Tool calls: none.
