# Card 5 — runfsm-tracked-reads

**Create:**
- `internal/runfsm/read_test.go`
- `plan:internal/runfsm#Read` -> `func Read(location *lymcwd.Location, slug string) (Records, error)`
- `plan:internal/runfsm#Owner` -> `type Owner struct`
- `plan:internal/runfsm#OwnerOf` -> `func OwnerOf(boardDir, slug string, parentOf func(slug string) (string, error)) (Owner, error)`
- `plan:internal/runfsm#TestRead` -> `func TestRead(t *testing.T)`

**Extends:**
- `internal/runfsm/doc.go`

**Uses:**
- `plan:internal/runfsm#Records`
- `internal/shedengine#Status`
- `internal/boardengine#ReadTasks`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#Owners.AdoptSubject`

**Intent:** Fill `Records` from tracked records alone, with every `.lym` cache absent, and read the owner input.

`Read` takes any worktree of the hub and resolves the prime itself through `fabricengine.PrimeName`.
- The board entry comes from `boardengine.ReadTasks` over `fabricengine.BoardDir`.
- The batten record is the prime's `shedrun.StatusFile` of the slug, decoded with `shedengine.ParseStatusTolerant` from the file's bytes; no lock is taken, so no `.lym` path is read or created.
- The child record is the pair's own `shedrun.StatusFile(pair, shedrun.SelfRunID)` when this machine has the pair.
  Without the pair it is the pair's records branch on origin: the task branch through the topology's `PairTaskBranchOrOrigin`,
  its records branch through `fabricengine.RecordsBranchName`, and the file through `fabricengine.RecordsBranchFile` at `shedrun.TreeRunFileRels`' status path.
- `TaskBranchOnOrigin` is read through the topology's `TaskBranchOnOrigin`, only for a cancelled batten record.
- `PastParentMerge` is read only for a child at Finalize, and without the pair as well as with it:
  the parent is `hubgeom.ParentOf`, the prime's current branch when it names the prime, else the beam's task branch through `PairTaskBranchOrOrigin`;
  `fabricengine.Open(prime).HoldsWork` judges origin's remote-tracking parent branch against origin's remote-tracking task branch, as this machine last fetched them.
  The local parent branch never decides it, except under the push skip of `fabricengine.EnvSyncOptions`, where the local branches decide.
- `PairPresent` is whether the pair's task worktree is on disk.
Every git read goes through `fabricengine`; runfsm runs no git itself.

`Owner` holds `Here`, `Owner` and `Subject`.
`OwnerOf` reads `machinename.Name`, then `boardengine.ReadOwners(boardDir)` with `Owners.Effective` and `Owners.AdoptSubject` over `parentOf`, as battencli's cancel reads it today; it reads the local Bolt and pulls nothing.

`doc.go` gains the read paragraph: which record comes from where, and that origin's records branch stands in for a pair this machine lacks.
`read_test.go` is tier1, on hub fixtures from `hubforge`:
`TestRead` covers a child read from the pair, the same child read from origin's records branch once the pair is gone here,
a `landing` run whose local parent branch holds the task's work while origin's fetched parent does not reading not past its merge,
the same run once origin's parent holds it reading past its merge, a cancelled record whose task branch is gone reading `queued` through `Derive`,
and every state reading the same after the prime's and the pair's `.lym` directories are removed.

**ImpactSummary:** Adds the read paragraph to runfsm's doc; runfsm has no importer yet.

**Commit:** 5: runfsm-tracked-reads
