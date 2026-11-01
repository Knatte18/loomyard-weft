```yaml
lens: way-forward
outcome: reported
```

Task: way-forward lens review of the plan's refusal, stop and recovery ends.

**Findings**

1. MEDIUM — card 9, D6 seam-failure warning. `PullResult.SeamWarning` is added, but none of the code that prints a pull result outputs it, and card 9 lists none of that code. So for a failed take-in render, the way forward (`lym heddle render`) ends up only in the log.
   - `internal/fabriccli/weft_verbs.go:472-484`: `pullResultMap` copies an explicit field list into `lym fabric pull`'s envelope, and the new field is not among them.
   - `internal/fabricengine/hubpull.go:141-153`: `pullPairSide` reduces the result to `HubPullMoved`/`HubPullUpToDate` and drops the warning.
   - `internal/battencli/land.go:89` also calls `fab.Pull`.
   - D6 says the move's result carries the warning. This is the carry-over risk `detached-seam-failure-reaches-no-one` showing up in foreground verbs.
   - Fix: add `seam_warning` to `pullResultMap`, put the warning in the hub-pull side's `Detail`, and list those members on card 9.

2. MEDIUM — cards 13, 14, 17, 18: a `heddle.json` that does not decode has no working way forward.
   - Card 17 logs `lym heddle list` as the way to inspect the store. Card 18's `list`, `set` and `remove` all load the store through `Load`/`Decode` first, so each fails the same way.
   - Every render (seam or `lym heddle render`) fails too, and its warning names `lym heddle render`, which also fails.
   - PATTERN-heddle-instructions (card 1) makes the verbs and the replayer the file's only writers, so no sanctioned repair exists.
   - No card names what a decode refusal says, or a step that can succeed from that state, such as restoring the file from Bolt history with `git -C <Bolt checkout> checkout HEAD -- heddle.json` or a reset verb.

3. MEDIUM — card 12 against D7: "a claimed path with no replayer registered fails naming the registration".
   - Card 12 claims paths only through registered replayers (`claimantOf`), and its overview Shared Decision says "a path no registered replayer claims is cherry-picked".
   - So with heddle unregistered, a `heddle.json` commit is cherry-picked instead of failing. On a conflict, the stop's way forward is a manual `recordsConflictWayForward` resolution rather than naming the missing registration.
   - That D7 acceptance case ("a claimed path with no replayer fails naming the registration") has no row. `TestBoltReplayers` only empties the whole registry, which trips the ledger guard instead.

4. LOW — card 18, the `render` refusal for a prunable worktree.
   - It names `lym fabric list`, but that list still shows the prunable entry, so picking it again gives the same refusal.
   - From this state the step that progresses is pruning or removing the stale worktree, or naming a live one. The refusal text should say so beside `lym fabric list`.

5. LOW — card 14: the warning for a root `.heddle` that is a file, or a directory holding other files, names only the path.
   - It gives no next step, such as moving or removing it and then running `lym heddle render`.
   - Every file target stays unwritten until someone acts, and nothing says what to do.

6. LOW — card 18, the outside-hub refusal.
   - The pre-run goes through `RequireDrivableWorktree`, which also refuses the Bolt checkout and a weft sibling. Those sit inside a hub, yet the refusal tells the user to "run lym heddle inside a worktree of a hub", which misdirects them there.
   - Keeping `RequireDrivableWorktree`'s own text, or naming "a code worktree such as the prime", would progress.

7. LOW — card 18, `write`'s sync spawn.
   - The card does not say whether a `spawnSync` failure after a successful save is refused or ignored. The board ignores it (`internal/boardengine/board.go:162`).
   - If heddle reports it as a failure, the saved `heddle.json` is left uncommitted, and the message should name `lym board sync` as the way to commit and push it.

**Further tool calls**
- Grep for `CommitWritten`/`PullThenCommitWritten`/`fab.Pull` callers and pull-result printing: my context lacked whether `PullResult` fields reach any output, and whether a seam fire site runs under the Bolt lock.
- Read `weft_verbs.go:465-500` and `hubpull.go:85-160`: my context lacked how `lym fabric pull` and hub pull print a pull result.
