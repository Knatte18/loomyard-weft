```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal check of the review proposals carried out as proposed (F1, F2, F3, F6, F7, F11).

- **MEDIUM** — `internal/battencli/wire.go`, `Push` closure (F2 proposal)
  - **Summary:** the bench pairs are now removed with no prime lock held, although that removal writes the hub's worktree registry, which the producer's design serializes under that lock.
  - **Evidence:** `internal/battenshed/doc.go` says "Runs creating different pairs thereby overlap on everything but these steps: those that read the prime's tree or write the hub's worktree registry."
  - **Concrete case:** run A's teardown push runs `RemoveBenchPairs` → `td.Run` → `Topology.Remove` (worktree removals, branch deletions, workspace file rewrite) while run B's Worktree-Create holds the prime lock for its add, or run C's teardown holds it for its removal. Before the fix the bench removal was inside that exclusion.
  - **Possible mitigations I did not verify:** git or fabric locks may cover this, and `lym fabric remove` by hand appears to take no prime lock. The doc.go sentence is still not held.

- **LOW** — `internal/battenshed/deps.go`, `TeardownDeps.Push` doc (the sibling F2's fix key did not name)
  - **Summary:** the field still says only "Push brings the pair's commits to origin", and "Any other error is a refused push". The wired Push now first removes other pairs, and returns that failure as such an error.
  - **Evidence:** `deps.go:377-382`. Relatedly, the `wire.go` comment says "a pair already gone pushes nothing and reads level", but for a gone task pair Push still removes bench pairs. `Remove`'s "runs only after Shutdown, Push and Level succeeded" still holds.

- **LOW** — `internal/pairteardown/leftover.go`, the `EndSubjectProcesses(prime, slug, false)` call, and the `pattern/PATTERN-pair-remote.md` bullet (F1 and F11 proposals)
  - **Summary:** the borrowed function's contract is "every process this machine runs for a subject another machine now owns", and it logs "ended the processes of a subject another machine owns" for every bench removal. That log line is false for a bench pair.
  - **Evidence:** `internal/pairteardown/endsubject.go`, the doc comment and the `logger.Info` call. It also spares the roles dobby, adopt, sync, push and slack-relay, none of which matters for this case.
  - **Pattern gap:** the new bullet ends processes before any refusal probe. The same section says "a refused removal ends no owned process", and the bullet does not say it is an exception to that. A bench removal refused after the end (a held teardown lock on archive, or a refused tag push) leaves its processes killed.

- **LOW** — `internal/fabriccli/roleguard.go`, `benchPairAdmitted` using `hubgeom.PrimeWithoutGit` (F3 proposal)
  - **Summary:** on a repository whose refs are not files (`extensions.refStorage` other than `files`), every bench `add`, `remove` and push is refused. The guard now depends on a reader that rejects such repositories.
  - **Evidence:** `internal/dotgit/worktrees.go` returns `ErrRefStorage` "so a caller can ask git instead", and the guard does not fall back. The push path also drops that error, so the refusal gives no reason there.

F6 and F7: no case found. I did not check whether `lym fabric prune --apply` clears a half-removed pair whose worktree fails to resolve. `Prune`'s doc only promises removal of stale weft worktrees, and reports unregistered directories as Unowned.

Tool calls:
1. `grep` for `TeardownDeps` and `Teardown.Push`/`Remove` callers: the `deps.go` field docs were not in context.
2. `sed` of `prune.go` and a `grep` for prime-lock uses: Prune's contract was not in context.
