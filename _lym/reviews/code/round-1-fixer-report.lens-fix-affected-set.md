```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the sets behind F1 and F2. The sites below are in the same set as the review's findings and were not fixed.

Unchecked: F3, F4 and F5 have no set premise.

- **MEDIUM** — `internal/battencli/readoptstop.go:194-195`, `leftoverArchiveWayForward`. This is the same defect as F1 and F2 and sits in the same set: manual git commands run against the records store.
  - It prints `` `git -C <recordsRoot> push origin <tag>` `` with no destination, which F2 says bypasses the store's push refspec and writes into the code origin's `refs/tags/` in a one-repository hub.
  - It also prints `` `git push --force-with-lease=refs/heads/<branch>:<tip> origin --delete <leftover.Branch>` ``, a short-name delete with a `refs/heads/` lease, which F1 says cannot reach `refs/lym/heads/`.
  - It reads `leftover.Branch` and `leftover.RemoteTip`, and the fix added `originRef` to `ErrRemoteLeftover`, but this function does not use it. The tag push also ignores `originNamespaceAt(recordsRoot)`.
- **LOW** — `internal/fabricengine/remoteteardown.go:186`, the discard branch of `failedRemoteSteps`. It pushes `refs/tags/<tag>` from `job.Worktree`, which is the code store, so it is correct as written. This matches the review's "code-store command unchanged". No fix is needed, but a one-line note in the fixer-report would record that the set was checked.
- **LOW** — `internal/fabricengine/destroy.go:1096`. The message tells the operator to run `git push <remote> --delete <branch>` with a short name. `req.repoDir` is resolved through `originNamespaceAt` at `destroy.go:1623`, so it may be a records store. I did not check whether this refusal can be reached for a records branch. It is a candidate for the same F1 defect if it can.

Calls made:
1. `grep` for manual push and delete command strings in non-test Go files, to enumerate the set; my context held only the files the review named.
2. `sed` of `readoptstop.go:165-200` and `destroy.go:1085-1097`, to read the two non-review sites the grep surfaced.
