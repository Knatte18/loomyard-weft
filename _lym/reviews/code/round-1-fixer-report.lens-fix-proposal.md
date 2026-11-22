```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. Task: challenge each proposal the fixer carried out as proposed (F1–F7), looking for a concrete case where it is wrong.

Findings:

1. **LOW**: F5's appended clause lands after the way forward.
   - Text: `destroy.go`, in the `fail` closure: `cause = fmt.Errorf("%w; the records branch was returned to what origin held before", cause)`.
   - The proposal appends inside `fail`, which runs after `seedPushCause` has built "…; way forward: run the same clone again".
   - The message then ends "way forward: run the same clone again; the records branch was returned…", so a reader can take the restore note as part of the way forward.
   - The also-failed branch already did this, so it is consistent, but it is a new misreading for the success case.
   - Moving the note before `seedPushCause`, in the board branch, would avoid it. I found no wrong behaviour, only wording.

2. **LOW**: F6's fetch-failure message may name the wrong fault.
   - Text: the `cannotCompare` closure in `refuseUnseededBoard`, applied to `j.repo.Fetch()`.
   - `ls-remote` has just succeeded, so a `Fetch` failure is most likely a transient transport error or a remote-refs lock, not a records store that does not read.
   - The way forward "run the same command again once the records store at %s reads" points at the wrong cause, though a rerun still helps.

3. **No finding**: F2's "shares history means this hub's seed" rule (`sharesHistory`).
   - Case built: two hubs' boards could share a root commit if the seed commit were deterministic.
   - A grep of the non-test code in `fabricengine`, `gitrepo`, `gitexec` and `hubforge` found no pinned `GIT_*_DATE` on the board's seed commit (the only hit, `integrateweft.go:381`, is unrelated), so separate `git init` boards get distinct roots.
   - I did not read the seed-commit code itself. Two hubs' boards could only collide on a same-second, same-content, same-author seed commit, which is a remote case.

4. **No finding**: F4's reuse of `ErrLeaseRejected`.
   - Wrapping it with `%w` while the original error goes in with `%v` makes `seedRefusedByOrigin` (an `errors.As` on `GitError`) return false, so the join way forward is reached.
   - Case: `withdrawn=false` with a nil error occurs only when origin's listed tip differs from both the pushed tip and the lease. An absent board matches the lease of `""` and keeps the hub, so no hub is torn down wrongly.

5. **No finding**: F2's added `Fetch` and F3's new rows.
   - `Fetch` takes only the remote-refs lock. `retryBoltPush` already calls `repo.Fetch()` while holding the push lock, so I found no deadlock.
   - The descendant row fails with the fix absent: without the fetch, `originTip` is missing locally and the old code reports another board.

Tool calls:
1. `grep` for pinned commit dates and seed-commit code in `internal/fabricengine`, `internal/gitrepo`, `internal/gitexec` and `internal/hubforge`. Why: my context did not show how the board's first commit is created.
