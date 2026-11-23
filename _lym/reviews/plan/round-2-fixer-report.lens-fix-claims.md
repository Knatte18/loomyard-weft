```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checked every sentence the fix added to cards 1 and 10 against the code at HEAD.

**[MEDIUM] Card 1: after the delete, the next push of that branch fails at the upstream read instead of skipping**

- **Changed text:** card 1, the new `DeleteRemoteTrackingRef` bullet. It says the branch "keeps its upstream setting and reads as upstream-set with a pruned ref", and gives as the reason not to use `DropRemoteTracking` that "its next push passes `checkOrigin`'s no-upstream branch and recreates the branch origin deleted."
- **Summary:** keeping the upstream does stop the branch from being recreated. But the next push in that state does not reach the ended-entry skip; it fails at `checkOrigin`'s first step with an error that is not `ErrNoUpstream`. The card states no behaviour for that push.
- **Evidence:**
  - `internal/gitrepo/blobread.go:331-333`: when the upstream is set and mapped but the tracking ref is absent, `UpstreamSHA` returns `"gitrepo: resolve upstream %s: %w"` and not `ErrNoUpstream`.
  - `internal/fabricengine/pushclassified.go:626-630`: `checkOrigin` returns `false, upstreamErr` for any error other than `ErrNoUpstream`, before `awaitOrigin` and the `view.ended` skip at line 641.
- **Where it bites:** card 10's teardown, when `Level` reads false because a records tip moved and the push runs again over a code branch whose ref card 1 deleted. That push returns a plain error, which card 10 treats as Stuck, not as the "pushes with no error" its line 49 assumes.
- **Possible remedy:** have card 1 make `checkOrigin` treat an upstream that is set but whose tracking ref is absent like the ended skip, or have it say how that error is handled.

**Claims that hold:**
- Card 10: `NilSeams` skips `nilLegal` paths, so an `intentionallyNil` entry for such a path fails the test's second loop (`internal/loomcli/wiring_seams_test.go`, the second `for` loop).
- Card 2 reads a branch with no upstream as ahead and not held, and a branch with a set upstream and an absent ref as level and held (`_lym/plan/02-*.md` lines 27-28 and 46-47). So the card 1 claims that depend on card 2 hold.
- `resolvePushDestination`'s tracking ref comes from the remote's fetch refspecs, mapped through `ns` (`internal/gitrepo/pushdestination.go`), as the new bullet says.
- `DropRemoteTracking` does unset `.remote` and `.merge` (`internal/gitrepo/push.go:215-221`).

**Tool calls:**
1. Grepped `checkOrigin` in `internal/fabricengine` and the upstream and held lines of card 2, because neither was in my context.
2. Grepped `UpstreamSHA` and `ErrNoUpstream` in `internal/gitrepo`, to see what an absent tracking ref with a set upstream returns. My context lacked that.
