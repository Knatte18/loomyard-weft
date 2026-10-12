# Card 25 — batten-lock-verbs

**Create:**
- `internal/battencli#lockCommand`
- `internal/battencli#lockStore`
- `internal/battencli#runLockList`
- `internal/battencli#runLockRelease`
- `internal/battencli/lock_test.go`

**Edit:**
- `internal/battencli#Command`
- `internal/battencli#battenCLI.resolvePersistentPreRun`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/battencli#RefuseUnlessPrime`
- `internal/parentlock#Store.List`
- `internal/parentlock#Store.ForceRelease`
- `internal/parentlock#Store.ReleaseAll`

**Intent:**
Add the parent lock's verb group (D11), so the orch and the operator can see every parent lock and free one whose owner is alive but wedged.
`Command` mounts `lock` under `batten`, with `list` and `release`, each with an `audience` of operator, a `Short`, and JSON envelopes (`PATTERN-cli-cobra`).
The `lock` group runs its own pre-run: `resolvePersistentPreRun` skips the slug wiring for the `lock` subtree, and `lockStore` resolves the cwd, refuses outside the prime through `RefuseUnlessPrime` as every batten verb does, and builds the store at `parentlock.Dir(fabricengine.BoardDir(hub))`.
- `lyx batten lock list` prints one entry per record: branch, owner slug, pair, run id, pid, liveness, age from `acquired_at`, and the queued waiters oldest first; an unreadable record lists with its error.
- `lyx batten lock release <parent-branch>` force-releases that branch's record; a branch with no record refuses with `no parent lock on "<branch>"; way forward: run "lyx batten lock list" to see the held locks`.
- `lyx batten lock release --all` releases every record and names them; a branch argument beside `--all`, or neither, refuses naming the two forms.
Bound: a release frees the record only and never stops a run; the released run reacquires at its next lock-taking row and reroutes if the parent moved meanwhile.
`lock_test.go` runs the verbs against a temporary store: list output fields, release of one branch, the unknown-branch refusal, `--all`, the argument refusals and the non-prime refusal.
`refusal-spec.md` gains the three refusal rows, and `docs/overview.md`'s batten line names the verbs.

**ImpactSummary:** a new `lyx batten lock` verb group; the existing batten verbs and their pre-run are unchanged except for skipping the new subtree.

**Commit:** 25: batten: lock list and lock release verbs
