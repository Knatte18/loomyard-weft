```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F5 change set for twins left unchanged (fix-consistency lens).

**Findings**

1. **MEDIUM** — `internal/battencli/readoptstop.go:194-195`, `leftoverArchiveWayForward`. This is the twin of F1 and F2, and it was left as it was.
   - It handles the same `*ErrRemoteLeftover` for a records branch and builds the manual steps for a records store.
   - The tag push is `git -C <recordsRoot> push origin <tag>`, with no destination. This is the exact pattern F2 fixed in `failedRemoteSteps`.
   - The branch delete is `push --force-with-lease=refs/heads/<Branch>:<tip> origin --delete <Branch>`. It uses the short name and a `refs/heads/` lease. This is the pattern F1 fixed for the weft message.
   - In a one-repository hub the tag lands in the code origin's `refs/tags/`. The delete and its lease both miss `refs/lym/heads/<Branch>`.
   - `leftover.originRef` is unexported, so `battencli` cannot read it. The fix would need an exported accessor or field, or a namespace read of `recordsRoot`.
   - The review's `lens:code-affected-set` named only the two sites that got fixed, so this is the same affected-set gap.

2. **LOW** — `internal/fabricengine/remoteleftover.go`, `ErrRemoteLeftover` and the `originRef` field. A zero-value or externally constructed weft-kind error carries no `originRef`.
   - `leftoverDivergedAdoptedWeft` is the zero `kind`.
   - `battencli/owner_readopt_test.go:135` builds `&ErrRemoteLeftover{Slug, Branch, RemoteTip}`. That is a weft-kind error with an empty `originRef`, so `Error()` would print `git push origin --delete ` with no ref.
   - A search for `Error()` calls on these errors found none in the `battencli` tests, so nothing fails today. The exported-struct contract still silently depends on a field that only `probeLiveLocalWeft` sets.

No findings for the other twins I checked:
- The discard-path tag push at `remoteteardown.go:186` is a code-store command, which F2 says to leave alone.
- The warp message in `Error()` keeps its short name, which is correct for the code store.
- `destroy.go:1096` prints a `git push <remote> --delete <branch>` command. It runs against `req.repoDir` and `req.remote`, and I did not verify which store that is. I'm noting it without a finding.
- F5's `doc.go` reword now matches the sentence under it, and the removed trailing sentence is not duplicated.
- The F4 row matches the line shape `pushArchiveTag` sends.
- The F3 message uses `OneRepositoryBoardBranch`, consistent with the keys written just above it.

Calls made
- `grep` for manual delete and push commands across `internal` and `docs`: to find twins of the changed messages, which my context lacked.
- `sed` of `readoptstop.go:165-200` and `destroy.go:1070-1096`: to read the two hits the grep surfaced, which my context lacked.
