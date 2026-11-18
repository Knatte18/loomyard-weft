```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I tested each proposal the fixer carried out against a concrete case.

**MEDIUM: the F9 proposal can stall the row with a dirty tree and an idle agent**
- Changed text: D4, "It stops the session first and re-reads the tree, as it does for the continue flag. On a tree dirty before or after the stop it keeps waiting, with the session up for the agent to commit or discard, and does not reroute."
- Case:
  1. The verb sees a clean tree and tells the agent to end its turn.
  2. The agent edits a file anyway and goes idle.
  3. The mark is set but the tree is dirty, so the row keeps waiting. No one tells the idle agent to commit or discard, and the row waits until the parent steps in.
- Contradiction: if the tree turns dirty after the stop, the session is already down, yet the sentence says "with the session up".
- Why the continue-flag parallel fails: under the continue flag the agent wrote the flag itself and is still working. Here the verb has told it to end its turn.
- Evidence: `internal/landingshed/prreview.go:684-691` (`sessionStopWanted` stops only on a clean tree at the gated head and otherwise "the session is left at work"). The continue-flag path relies on a live agent that knows to clean the tree. Neither D4 nor its new dirty-tree criterion says what reaches an agent that ended its turn, or the parent.

**LOW: the F7 fix-here count is not scoped to the episode the text names**
- Changed text: D4 Send back, "added to those the verb answered fix-here earlier in the PR-Review episode"; Bound, "one episode fixes at most `pr_review_small_max` findings in place".
- Problem: the count lives in the pull-request record and is cleared only on the send-back reroute ("On a clean tree it clears the mark, … and its fix-here count").
- Case: PR-Review is entered again by any route other than this reroute. Fix-here findings from the earlier episode still count, so a small set can trigger a send-back against a threshold the text says is per episode.

No findings on F1, F2, F3, F5, F6, F8, F10 or F11:
- F1: keying on the trailers re-rules every listed file, and the fix states that cost in D3's Bound.
- F2: the loom scratch directory already holds the Parent-Align fixer report and merge records, and no new generation archives it.
- F6: the new parent ask names the new session.

Tool calls:
- One Bash call read `prreview.go` around 280–300 and 680–700 and grepped `LoomScratchDir`/`RemoveAll`. My context lacked the continue-flag dirty-tree handling and the scratch directory's lifetime.
