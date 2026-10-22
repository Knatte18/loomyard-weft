# Card 14 — lyx tender status lists the owned processes and the orphans

**Create:**
- `plan:internal/tenderengine#ProcessReport` -> `type ProcessReport struct`
- `plan:internal/tenderengine#OwnedProcess` -> `type OwnedProcess struct`
- `plan:internal/tenderengine#readProcesses` -> `func readProcesses(deps Deps, processes []proc.Process, processesErr error) ProcessReport`
- `plan:internal/tenderengine#orphanOwner` -> `func orphanOwner(owner string, records []runproc.Record, worktreeNames map[string]bool) bool`
- `plan:internal/tenderengine#unregisteredOrphans` -> `func unregisteredOrphans(processes []proc.Process, hubPath, executable string, worktrees []string, registered map[int]bool) []StaleProcess`

**Edit:**
- `internal/tenderengine#Deps`
- `internal/tenderengine#StatusReport`
- `internal/tenderengine#Status`
- `internal/tenderengine#TestStatus`
- `internal/tendercli#HubDeps`
- `internal/tendercli#statusCmd`
- `internal/fabricengine#Topology.lockPairTeardown`
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/runproc#Owners`
- `internal/runproc#Record`
- `internal/runproc#HubOwner`
- `internal/runproc#RoleBatten`
- `plan:internal/fabricengine#RunProcDir`
- `internal/tenderengine#staleProcesses`
- `internal/tenderengine#fakePair`

**Intent:**
D8's reading.
`Deps` gains `RunProcs func() (map[string][]runproc.Record, error)` and `Worktrees func() ([]string, error)`, the paths of every worktree git still registers, prime and task alike; nil reads none.
`StatusReport` gains `Processes ProcessReport` (`processes`): `Owned []OwnedProcess` (`owner`, `role`, `pid`, `age_seconds` from `RegisteredAt` to `deps.Now()`, `orphan`), `Unregistered []StaleProcess`, `UnregisteredSupported bool` and `Error string`.
`Status` reads the process listing once and hands it to both the replaced-binary part and `readProcesses`.

`orphanOwner` reads an owner's records as orphans when the owner is a slug, never `runproc.HubOwner`, that has no pair, read as no path `Worktrees` returns whose base name is the slug, so a worktree git still registers keeps its slug's records out of the orphans even when it is prunable or does not resolve and `HubDeps` lists no `Pair` for it, and whose batten is not running, read as D4 says: a `running` batten record counts only while a record of the slug with role `batten` is alive, and `RunProcs` lists only live records.
Within an orphan owner, a record whose pid's working directory in the same process listing lies inside a path `Worktrees` returns is listed with `orphan` false, so D8's bound that reap never ends a process working inside a registered worktree holds for registered records too; where the listing is unsupported no record is excluded on that ground.
`unregisteredOrphans` lists each process running a lyx executable (the running lyx's file name, as `staleProcesses` matches it) whose working directory is a deleted directory (the kernel's ` (deleted)` suffix) inside the hub and outside every path `Worktrees` returns, whose pid no record registers, and whose exe is not deleted, since those are the replaced-binary part's.
A lyx process in a deleted directory inside a live pair's worktree is therefore not listed.
On a platform that cannot list processes the owned records are still listed with their pids and `UnregisteredSupported` is false, as the replaced-binary part reports; a failed `RunProcs` or `Worktrees` read lands in `Error` and the rest of the report stands.

`HubDeps` fills `RunProcs` with `runproc.Owners(fabricengine.RunProcDir(hub))` and `Worktrees` with the paths of every entry of the `fabricengine.List` reading it already makes, prunable ones included.
`statusCmd`'s help names the processes part.
`fabricengine`'s `lockPairTeardown` `ErrPairPushRunning` refusal, which a push still held under a hub pause or a parent-locked push can still raise, names the stuck push child's pid as `lyx tender status` now lists it under the pair's owner, in place of "whose pid its spawn logged"; no test asserts the old wording.

`TestStatus` gains rows over fake seams: a slug owner with a registered worktree is listed with `orphan` false, also when that worktree has no `Pair`; a failing `RunProcs` and a failing `Worktrees` each land in `processes.error` with the rest of the report standing; a slug owner with no pair and no live batten record is listed orphan; one with no pair but a live `batten` record is not; a record of an orphan owner whose listed process works inside a registered worktree is not; a `HubOwner` record is never orphan; an unregistered lyx process in a deleted hub directory outside every worktree is listed, one inside a live pair's worktree is not, one outside the hub is not, one whose pid is registered is not, and one running a deleted exe is left to the replaced-binary part; `Processes` returning `errors.ErrUnsupported` keeps the owned list and reads `unregistered_supported` false.
The docs' `# Status` sections and the overview's tender line name the part.

**ImpactSummary:** two nil-safe Deps seams and one report part; every existing Status caller (tender, hibernate) reads the report unchanged.

**Commit:** 14: lyx tender status lists owned run processes and orphans
