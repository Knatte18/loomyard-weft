# Card 3 — runfsm-reader-batten-run

**Create:**
- `internal/runfsm/read.go`
- `internal/runfsm/read_test.go`
- `plan:internal/runfsm#Read` -> `func Read(prime *lymcwd.Location, slug string) (Records, error)`
- `plan:internal/runfsm#Owner` -> `type Owner struct`
- `plan:internal/runfsm#ReadOwner` -> `func ReadOwner(prime *lymcwd.Location, slug, here string) (Owner, error)`
- `plan:internal/runfsm#TestRead_EveryStateFromTrackedRecordsAlone` -> `func TestRead_EveryStateFromTrackedRecordsAlone(t *testing.T)`
- `plan:internal/runfsm#TestRead_PastParentSideMergeReadsOriginsFetchedParent` -> `func TestRead_PastParentSideMergeReadsOriginsFetchedParent(t *testing.T)`
- `plan:internal/battencli#TestBattenPreRun_RefusesWhatTheActRuleHolds` -> `func TestBattenPreRun_RefusesWhatTheActRuleHolds(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.wire`
- `internal/runfsm/doc.go`

**Uses:**
- `internal/shedrun#StatusFile`
- `internal/shedrun#TreeRunFileRels`
- `internal/fabricengine#AnchoredFileAt`
- `internal/fabricengine#Fabric.HoldsWork`
- `internal/fabricengine#ReadOriginFor`
- `internal/boardengine#OpenHub`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#Owners.Effective`
- `internal/boardengine#Owners.AdoptSubject`
- `internal/boardengine#OwnedElsewhere`
- `internal/boardengine#IsDoneStatus`
- `internal/hubgeom#ParentOf`
- `internal/battencli#battenCLI.refuseCovered`
- `internal/battencli#battenCLI.thisMachine`
- `internal/battencli#battenCLI.refuseOwnedElsewhereOnLocalBoard`
- `internal/battencli#TestBattenPreRun_ContinuePass`
- `internal/battencli#TestBattenRun_RefusesWhileAPauseCoversTheSlug`

**Intent:**
Add runfsm's reader and make `lym batten run` its first caller, so the package joins the production graph with every function reached (PATTERN-test-only-code).

`Read` gathers `Records` for slug from tracked records alone, with every `.lym` cache absent:
the batten record is the prime's `shedrun.StatusFile(prime, slug)`, read raw and decoded with `ParseRunRecord`;
the board status and release come from the hub's Bolt board (`boardengine.OpenHub`), released meaning what the dobby's start reads, a task entry with an empty status whose parent is this worktree;
the child record is the pair's own `status.json` when this machine has the pair, and otherwise the same file read with `fabricengine.AnchoredFileAt` at origin's records branch of the pair, the branch named by the slug's origin record (`fabricengine.ReadOriginFor`), so a machine without the pair reads what origin holds (D1);
whether the task branch is on origin, and whether origin's fetched parent branch holds the task branch's work.
The parent-side merge is judged with `Fabric.HoldsWork` on the remote-tracking refs of the parent branch and the task branch as this machine last fetched them, never the local parent branch and never the pair's checkout, so it needs no pair here; this settles the carried-over `readrun-pastparentmerge-no-pair` finding.
Under the push skip, which pushes nothing to origin, the local parent branch decides it instead (D1).
It is read only when the child record is at `Finalize` or past it, so other runs pay no git call.
A missing record reads as absent; an unreadable one is an error naming its path.

`ReadOwner` reads the slug's effective owner and adopt subject from the local Bolt (`boardengine.ReadOwners`, `Owners.Effective` and `Owners.AdoptSubject` with `hubgeom.ParentOf` as the parent lookup) and says whether it is another machine than `here` (`boardengine.OwnedElsewhere`); a caller that needs origin's owner fetches the Bolt first.

`battenCLI.battenPreRun` asks the act rule right after its run-status check and before the owner preflight, through a new seam field on `battenCLI`, set in `battenCLI.wire` to `runfsm.Read` and `runfsm.ReadOwner` with this machine's name, `Inputs.PauseCovers` from the pause coverage `refuseCovered` already read, and `Aside{ShedPause: true}`; a nil seam asks nothing, as the reconcile seam does.
A `wait` or `hand_over` refuses with `Verdict.Refusal(slug, "lym batten run <slug>")` before anything is written; a done run keeps today's run-status handling, since the act check runs after it.
This adds a refusal only: it skips nothing and lets nothing through that ran before.

`internal/runfsm/read_test.go` holds tier1 tests on a `hubforge` hub: `TestRead_EveryStateFromTrackedRecordsAlone` commits each state's records, removes the worktree's and the hub's `.lym`, and reads the same state, a released task with no record and a board status `done` reading `landed`, and a kept cancel's record reading `cancelled` from a clone without the pair;
`TestRead_PastParentSideMergeReadsOriginsFetchedParent` merges the task's work into the local parent branch only and reads the run as before its merge, then pushes and fetches and reads it past.
`TestBattenPreRun_RefusesWhatTheActRuleHolds` is a tier0 test with the seam faked: a `wait` and a `hand_over` verdict each refuse naming the way forward and write no status, and `act` goes on.
The doc gains the reader's paragraph.

**ImpactSummary:** `lym batten run` refuses a run the act rule holds before it writes anything; no other caller yet.

**Commit:** 3: runfsm reads a run from tracked records, and lym batten run asks its act rule
