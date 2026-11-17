```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge each proposal the fixer carried out as proposed, by building cases where it is wrong.

Proposals checked: F1, F2, F3, F4, F6, F7, F8, F9, F10, F11. F5 is left out: the fixer took one of its two options, accepting the bound, and the acceptance criteria still match that choice.

1. **MEDIUM.** D2's new paragraph says "A parent's answer reaches round N only as an `Added after Discussion` entry", and D3 says "Only the added entries D2's block lists answer a held finding; a `ruling` disposition does not." Both come from the F1 and F6 proposals. **Case:** the parent answers a held `decision`-class finding with a ruling sent as a creel message. The fixer carries the ruling out and records `ruling`. No `lym loom decision add` follows. In round N the block lists no entry for that finding, so D3 forces it `open`, it is carried, and the Bouncer escalates it again even though the ruling was already applied. The same happens when the parent answers through `lym loom decision replace`, which folds the answer into the main decisions and adds no entry. **Evidence:** `instruction-fix.md` "Parent ruling rule" says a ruling is "a message from your parent, or a decision notice". The shuttle `prompt.md` says to "change or withdraw an earlier decision … with `lym loom decision replace`". Neither path writes a new added entry, so the round loops on a question the parent already settled.

2. **MEDIUM.** D3's last paragraph says "`lym burler validate-review` resolves its worktree as the other burler verbs do". This is the fixer's way of carrying out F3. **Case:** today validate-review skips resolution: "it reads one told path and needs no hub, mode or engine wiring … Neither path needs a git repository" (`internal/burlercli/cli.go:66-69`). If it resolves the way the other verbs do, a review file outside any repository or hub fails in pre-run. It never reaches the promised path, "A file outside a run directory gets the shape checks alone". The way forward the text names cannot be reached in that state. The resolution should be scoped to the builder, attempted only when the file sits in a run directory, and the pre-run skip kept.

3. **LOW.** D3's added line "With no changes file a carried finding keeps its earlier location path and may not move it" comes from F9. **Case:** the snapshot cache is gone and round N-1's fix moved a MEDIUM defect to another file. The reviewer cannot carry it at the defect's new place. Marking it `outdated` and raising it as new is refused unless it is `BLOCKING` (D4: "the BLOCKING rule holds alone"). So the only legal record points at a location where the defect no longer is, and the fixer works from a stale location. **Evidence:** D3's carries rule against D4's no-changes-file rule.

4. **LOW.** D5 says "It records titles, as `writeDecisionSnapshot` does" (from F2). **Case:** a `replace` folds an entry titled "X", and the parent then adds a new answer that is also titled "X". The new entry matches the stored title, so it is never listed and the held finding stays open. **Evidence:** the multiset match in `addedSinceSnapshot` (`internal/shedadapters/bouncerdecision.go:336-347`). The count it replaces failed in more cases than this, so the titles are still the better choice; the gap is only that the record does not mention this edge case.

The other proposals hold:
- **F4:** a re-judge or replay reads the recorded post-fix id.
- **F7:** a round after a skipped fixer renders the block with empty dispositions.
- **F8:** the base parse already tolerates unknown keys.
- **F10 and F11:** wording-only changes; the "Review" acceptance bullet still points at the regression rule, which remains in D4.

No further tool calls; everything above came from what my context already held.
