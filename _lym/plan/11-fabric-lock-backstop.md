# Card 11 — fabric-lock-backstop

**Create:**
- `plan:internal/fabriccli#queueBackstop` -> `func queueBackstop(location *lymcwd.Location, boltDir, branch string) error`
- `plan:internal/fabriccli#TestAcquireBranchLock_WritesTheLandingsAStandingMarkOwes` -> `func TestAcquireBranchLock_WritesTheLandingsAStandingMarkOwes(t *testing.T)`

**Edit:**
- `internal/fabriccli#acquireBranchLock`

**Extends:**
- `internal/fabriccli/doc.go`

**Uses:**
- `internal/fabriccli#runHubPull`
- `internal/landingqueue#Store.Backstop`
- `internal/runpusher#ParentTarget`

**Intent:** Run D6's backstop at the fabric verbs' taking of the parent lock, so a move a dead leader made gets its ledger landings before anything else acts on the parent.

`acquireBranchLock` is the one place `lym fabric anchor` (through `AcquireParentLock`) and `lym fabric pull --hub` (through `runHubPull`'s pair lock) take the parent lock, so one call there covers both.
Right after `store.Acquire` it calls `queueBackstop(location, boltDir, branch)`.
An error releases the lock and is returned, naming the mark file and the cause, with the way forward "clear the cause the error names, then run the verb again, which writes the mark's landings before it acts": a ledger write or ref read that fails keeps the mark, since dropping it would lose the landings the parent owes.
`queueBackstop` builds `landingqueue.Store{Dir: landingqueue.Dir(boltDir), Branch: branch}` and runs `Backstop` with seams over the pair at `location`: `ParentHolds` as `IsAncestor(commit, "refs/heads/"+branch)`, `UpstreamLacks` as the negation against the branch's upstream ref, and `WriteLanding` as `runpusher.RecordLanding` on `runpusher.ParentTarget(prime, location)` with a `pushledger.Landing` of the slug, tip, records commit, branch and the time now.
A queue with no mark returns at once.
The fabric verb takes no queue entry and prunes none, since only a run's Finalize prunes, after its own backstop.
`internal/fabriccli/doc.go` gains a sentence that the verbs taking a parent lock write the ledger landings a standing landing-queue mark owes first.

`TestAcquireBranchLock_WritesTheLandingsAStandingMarkOwes`, tier1 in `package fabriccli`, over a `hubforge` hub whose parent pair holds two landing commits origin lacks and a standing mark naming them, rows: taking the lock writes one ledger landing per commit with its mark's records commit and removes the mark; a mark whose commits the parent does not hold is removed with no landing written; a push ledger made unwritable through `permkit.Chmod` returns the error naming the mark, keeps the mark and leaves the lock free.

**ImpactSummary:** One function whose two callers (anchor and hub pull) both gain the backstop, plus one doc sentence; nothing else in fabriccli changes.
