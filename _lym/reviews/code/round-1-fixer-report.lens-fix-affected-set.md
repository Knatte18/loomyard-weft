```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix's change set.

**Unchecked findings:** none. F1, F6 and F2/F8 were checked by search; F3, F5, F7 and F9 from context. F4 was not fixed.

**Findings**

1. **LOW** — `internal/fabricengine/pendingteardown.go`, the `PendingTeardown` type doc, which sits beside the new `SettleGonePairTeardown`.
   - The doc says the child removes the record, and that an unfinished teardown "Either leaves the pair's branches on origin."
   - The fix adds a second remover. It also handles the state F5 describes, a listed record whose pair is gone locally and on origin.
   - So the type doc is now incomplete about who removes a record, and its "branches on origin" sentence contradicts what `SettleGonePairTeardown` exists for. Only the file header was updated.

No other gaps in the sets:
- **F1:** `hubCancelDeps` and the test fake are the only places the two new seams need wiring. The other `lym batten cancel … --remove` strings are messages, not record re-runs. `fronttask`'s `ErrNothingLeft` handling still holds.
- **F2:** the battencli sentence is gone from `doc.go`.
- **F6:** no "four-argument" mention is left in `internal`, `pattern` or `docs`.
- **F8:** `pairteardown`'s remove closure is the only place `Request.Rerun` is forwarded.
- **Docs:** the cancel's help text is the only place my search found that lists its "nothing is left" refusal for users, and it is updated.

**Tool calls**
- grep for "four-argument" and `removeArgCount`: my context did not show the other mentions in the repo.
- grep for `Request{`, cancel `--remove` strings, `ErrNothingLeft` and `DiscardTaskBranch`: my context showed only the callers in the files already read.
- grep for "nothing is left" and "pending teardown" prose: my context did not show other docs describing these.
