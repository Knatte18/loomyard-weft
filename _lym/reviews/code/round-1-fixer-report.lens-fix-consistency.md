```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix change set (F1 to F9 commits) against its surroundings.

**Findings**

1. **MEDIUM.** Changed text: `internal/fabricengine/doc.go` ("what lym writes itself: ... the board's lock files and render manifest in the Bolt checkout"), the `written` list for the Bolt store in `hubWorkOriginLacks` (`clone.go`), and `leaveLymsIgnoredFiles`.
   - **Summary:** current code does not write `board.lock` or `board.push.lock` at the Bolt checkout's root, so the stated rule does not hold.
   - **Evidence:** `BoltLockPath` returns `lock.TrackedLockPath(boltDir, name)` (`junctionnames.go:70-71`), which is `<bolt>/.lym/locks/<name>` (`internal/lock/tracked.go:19-21`, `63-65`). Root copies are leftovers of older code, which is why the test has to add exclude lines for them.
   - **The twin left behind:** the other leftovers of the same kind (`.gitrepo-push.lock`, `board.json.swaplock`, present in the live `_bolt`) are still named as work. A plain `--reset` of that older hub is therefore still refused over lym's own empty files. Either word the skip as covering older hubs and say the other leftovers are still named, or treat all of them alike.

2. **LOW.** Changed behaviour: `RefuseHubWithoutBolt` now names the `--discard-work` clone and the work. Its twin paragraph at `internal/fabriccli/doc.go:33-36` was not updated.
   - **Summary:** it still says the refusal "names the `--reset` clone", and that "A plain re-clone into the same directory names it too, unless the hub holds work origin lacks".
   - **Evidence:** with store work and nothing using the hub, both refusals now name the same `--reset --discard-work` clone. Only `internal/fabricengine/doc.go` got the new sentence.

3. **LOW.** F1's third named edit was not made: `_lym/landing/summary.md:41` still reads "**Every ignored path counts as work.**"
   - **Summary:** the summary contradicts the narrowed rule now in `ignoredWork` and both `doc.go` files.
   - **Evidence:** line 41 and the lines below it say "a hub with build outputs refuses plain `--reset`" with no exception for the files lym writes. Line 12 likewise says ignored files "count as work".
   - If the fixer left it because `_lym` is outside its write surface, the fixer-report should say so.

Tool calls:
- A grep for `--reset` and `RefuseHubWithoutBolt` across source, docs and sandbox files: my context lacked the twins outside the files the fixer read.
- A read of `internal/fabriccli/doc.go:28-40`, the reconcile handler in `fabric.go` and parts of `_lym/landing/summary.md`: my context showed only single matching lines or earlier ranges of these.
