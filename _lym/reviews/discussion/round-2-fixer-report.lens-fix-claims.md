```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked every sentence the fix added or rewrote that states a fact about the code.

**[MEDIUM] D1: "A later `lym fabric checkout` on the prime moves the code and records worktrees and writes no config, so it cannot move the key."**
"Writes no config" is false.
When a records branch exists only on origin, checkout adopts it as a local tracking branch, and creating a tracking branch writes `branch.<b>.remote` and `branch.<b>.merge` config.
- `internal/fabricengine/checkout.go:221-224` says it "adopts the branch on origin as a local tracking branch".
- `switchOrForkWeft` (`checkout.go:230-247`) reaches that path through `resolveWeftBranch(rec, l, weftBranch, true)`.

The guarantee the sentence is there for still holds, because nothing in checkout writes `lym.primeRecordsBranch`.
The sentence should say checkout never writes that key, not that it writes no config.

**Claims that hold:**
- The board worktree sits on the code store's branch in a two-repository hub: `clone.go`'s `ensureBoardWorktree(weftPath, warpBranch, boardDir)`.
- Handing `ensureBoardWorktree` the board name adopts `origin/main` from a freshly fetched store.
  The function checks for a local branch first and a fresh init has none; it then reads `refs/remotes/origin/<name>`, which D1's fetch refspec `+refs/lym/heads/*:refs/remotes/origin/*` fills (`boardweft.go:24-53`).
- On a `--single-repo` store the board is orphan-created on `main`: no local or remote ref exists, so `boardweft.go:56-60` takes the orphan path.
- "Fails closed, as an unreadable board branch does today": `primaryWeftBranch` returns an error (`cleanup.go:339-350`), and the sweep (`cleanup.go:134-137`) and `resolveManagedBranch` (`destroy.go`) both refuse on that error.
- A two-repository hub reads the board's branch, as today: `cleanup.go:337-351`.
- `lym fabric checkout` moves the records worktree (`checkout.go:221-264`), which supports the support log's rejected alternative. "`git remote set-head` moves `origin/HEAD`" is standard git behaviour.

Further tool calls:
1. Grep of `checkout.go` for config and upstream writes. My context did not show what checkout writes.
2. Read of `checkout.go:195-290`. My context lacked `switchOrForkWeft`'s body, which I needed to judge the "writes no config" claim.
