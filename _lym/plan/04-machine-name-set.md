# Card 4 — machine-name-set

**Create:**
- `plan:internal/machinecli#nextFile` -> `func nextFile(cur current, name string) (machinename.File, int)`
- `plan:internal/machinecli#hubsToSweep` -> `func hubsToSweep(d deps, cwd string) ([]string, error)`
- `plan:internal/machinecli#liveHubs` -> `func liveHubs(dir string) ([]string, error)`
- `plan:internal/machinecli#hubOfDir` -> `func hubOfDir(dir string) (string, bool)`
- `plan:internal/machinecli#setName` -> `func setName(d deps, cwd, name string) (setResult, error)`
- `plan:internal/machinecli#setResult` -> `type setResult struct`
- `plan:internal/machinecli#hubOutcome` -> `type hubOutcome struct`
- `plan:internal/machinecli#sweepHub` -> `func sweepHub(d deps, hub, name string, replace []string) hubOutcome`
- `plan:internal/machinecli#recoveries` -> `var recoveries []string`
- `plan:internal/machinecli#fakeHubs` -> `type fakeHubs struct`
- `plan:internal/machinecli#TestNextFile` -> `func TestNextFile(t *testing.T)`
- `plan:internal/machinecli#TestHubOfDir` -> `func TestHubOfDir(t *testing.T)`
- `plan:internal/machinecli#TestName_Set` -> `func TestName_Set(t *testing.T)`
- `plan:internal/machinecli#TestName_SweepsHubs` -> `func TestName_SweepsHubs(t *testing.T)`

**Edit:**
- `plan:internal/machinecli#deps`
- `plan:internal/machinecli#productionDeps`
- `plan:internal/machinecli#newCommand`
- `internal/machinecli/doc.go`
- `docs/overview.md`

**Uses:**
- `plan:internal/machinecli#current`
- `plan:internal/machinecli#readCurrent`
- `plan:internal/machinecli#runName`
- `internal/machinename#CheckName`
- `internal/machinename#File`
- `internal/boardengine#SetOwners`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#OwnerWrite`
- `internal/boardengine#ApplySkipEnv`
- `internal/boardengine#LoadConfig`
- `internal/boardengine#New`
- `internal/boardengine#Board.UpsertTask`
- `internal/lymcwd#ResolveWorktree`
- `internal/preflight#BoardLymPresent`
- `internal/standalonestate#UserStateDir`
- `internal/state#WriteJSON`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#Bolt.Pull`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#BoltDir`
- `internal/fabricengine#BoardDir`
- `internal/fabricengine#ReadShortname`
- `internal/fabricengine#WriteShortname`
- `internal/hubregistry#Live`
- `internal/hubregistry#Record`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`
- `internal/hubforge#Hub.BoardDir`
- `internal/testkit/boardkit#InstallReplay`
- `internal/gitkit#Git`

**Intent:**
`lym machine name <name>` sets the name and sweeps owners (D5, D6).
`newCommand`'s leaf becomes `Use: "name [<name>]"` with `cobra.MaximumNArgs(1)`; with an argument it runs `setName(d, cwd, name)`, the cwd read through `lymcwd.CwdFrom`.

`deps` gains the hub seams, each a struct field filled by `productionDeps`:
`hubs func() ([]string, error)` (`liveHubs` over `standalonestate.UserStateDir`),
`cwdHub func(dir string) (string, bool)` (`hubOfDir`), and, per board directory `fabricengine.BoardDir(hub)`,
`pull func(boardDir string) (fabricengine.BoltSkip, string, error)` (`fabricengine.NewBolt(boardDir).Pull` with `fabricengine.NewMutations(filepath.Dir(boardDir))` and `boardengine.ApplySkipEnv`'s skip options, as `SetOwners` builds them),
`unpushed func(boardDir string) (bool, error)` (`Bolt.HasUnpushed`),
`readOwners` (`boardengine.ReadOwners`) and `setOwners` (`boardengine.SetOwners`).
The show never calls these, so a show test leaves them nil (card 3).

`liveHubs(dir)` returns the hub paths `hubregistry.Live(dir)` lists; its error names the record's path, `filepath.Join(dir, hubregistry.RecordFileName)`.
`hubOfDir(dir)` returns the hub `dir` lies in: `lymcwd.ResolveWorktree(dir)`'s `HubPath` when that resolves and `preflight.BoardLymPresent` holds for it, so a hub worktree and its subdirectories count;
else `dir` itself when `fabricengine.ReadShortname(fabricengine.BoltDir(dir))` finds a shortname, so the hub container directory counts;
else none, so a directory outside any hub adds nothing.
`hubsToSweep` returns `hubs`' paths plus `cwdHub(cwd)`'s hub, once each, in the registry's order with the cwd hub last when the registry lacks it.
A failed registry read refuses before any write with that error.

`setName` runs, in order, writing nothing before step 5:
1. `machinename.CheckName(name)`; an invalid name is refused with that error, which states the rule as its way forward.
2. `readCurrent(d)`.
   A locate error (a relative variable) is refused with it; no directory is refused naming `LOCALAPPDATA` on Windows and `XDG_CONFIG_HOME` or `HOME` elsewhere, with "set it to an absolute path" as the way forward.
   A read error that is not a decode error (the file exists but does not read) is refused naming the file's path, with the way forward: remove the file or fix its permissions, then re-run `lym machine name <name>`.
3. This machine's names: the valid current name, every non-empty `names` entry of a decoded file and the hostname rule's name when `hostName` yields one, a bad file included.
4. `hubsToSweep`. For each hub: read `unpushed` (kept for step 7), then `pull`; an error or a non-empty skip marks the hub `checked locally only`, with its detail, and the check goes on against its local Bolt.
   `readOwners` the board; a read error refuses the whole verb before any write, naming the hub and the error, with the way forward: repair that hub's board as the error says, then re-run `lym machine name <name>`.
   An unreadable board is refused rather than passed, since after a write the name is this machine's own and a re-run could never check that board again.
   When the prime owner or any entry's recorded owner equals `name` and `name` is not one of this machine's names, refuse naming the hub, the slug (or `prime`) and the owner: that name belongs to another machine.
   The refusal's way forward: choose another name; for a lost former name, write `{"name": "<name>"}` to the file's path by hand, and take back with `lym adopt` an owner left under another lost name (D5).
5. `nextFile(cur, name)` gives the file to write and how many empty entries it dropped:
   `name` becomes `name`; `names` starts from a decoded file's `names` (none for an absent or undecodable file);
   the old name (the valid current name, which is the hostname rule's name with no file) and then the hostname rule's name are each appended when absent;
   every empty entry is dropped and counted; `name` is then moved to, or appended at, the end.
   The file is written through `state.WriteJSON(path, lock, file)` only when no file exists, the file is bad, or the name or `names` differ from the decoded file's; a re-run with the name set and nothing to change writes nothing.
   The output says how many empty entries were dropped, whether an invalid `name` was dropped, and, for a file that did not decode, that its names were lost.
6. The replace set is the written `names` without `name`; empty entries are already gone.
7. `sweepHub` for each hub calls `setOwners(BoardDir(hub), OwnerRequest{Machine: name, Replace: replace})`, with no slug, prime, claim or `Takeover`, and reports one `hubOutcome`:
   an error is `failed` with its text, and its way forward is the error's own way forward when the error names one (as a dirty or diverged Bolt's or a deferred push's does), then the re-run `lym machine name <name>`; the other hubs go on;
   entries in `OwnerWrite.Lost` are reported each with the owner that stands and the re-run as its way forward, adding that `lym adopt` takes back an entry whose standing owner is another name;
   `Recorded` non-empty is `swept` with the slugs; nothing recorded is `pushed` when step 4 read the Bolt ahead of origin and `nothing to sweep` otherwise.

The result, `setResult` under `machine`, carries `name`, `file`, `names`, `written`, the dropped-entry and repair notes, each `hubOutcome` (hub path, checked-locally detail, outcome, slugs, lost entries, error, way forward), and `recoveries`.
`recoveries` holds D6's recovery lines as constant text: the owner pause the dobby's probe lifts after the sweep, `lym tender resume <slug>` for one it does not, the unpushed-commit case whose recovery is the re-run, `lym loom resume` in the pair for a halted Finalize, `lym beam start <slug>` or `lym beam start root` for a skipped or refused worktree, and lifting any pause with `lym tender resume` only after that hub's sweep reports done.
A failed or lost hub makes the verb exit non-zero through `output.ErrFields` with the same fields, its message naming the re-run; otherwise `output.Ok`.

Bound: the verb writes only the machine file at the path its seam hands in and, on each swept hub, owners recorded under this machine's former names (card 1's replace set); it never passes `Takeover` and never writes an owner recorded under any other name.
D5's bounds on owners pushed after the pull, former names another host took and shared names stand as the decision record states them.
D6's bound on a hub neither in the registry nor holding the working directory stands; the way forward is running the verb from that hub's container directory or one of its worktrees.

`internal/machinecli/doc.go` gains the set, the sweep, the bounds and the recoveries;
`docs/overview.md`'s **machine** bullet becomes `lym machine name [<name>]`: show, or set and sweep owners on every live hub and the hub of the working directory.

Tests:
- tier 0, `TestNextFile`, rows: `NoFileTakesTheHostnameNameAsOld` (host `osl-1046`, `osl46` gives `["osl-1046","osl46"]`), `AlreadySetChangesNothing`, `InvalidNameKeepsNamesAndAddsTheHost` (`{"name":"OSL!","names":["neve","osl46"]}`, host `box` gives `["neve","box","osl46"]`), `UndecodableKeepsOnlyTheHost` (`["box","osl46"]`), `EmptyEntryIsDroppedAndCounted`, `HandWrittenNameOnlyRecordsTheHost` (`{"name":"osl46"}`, host `box` gives `["box","osl46"]`), `NoHostnameAddsNone`, `NewNameMovesToTheEnd`.
- tier 0, `TestHubOfDir`, rows: `ContainerDirectoryIsItsOwnHub` (a directory whose `BoltDir` holds a shortname record written with `fabricengine.WriteShortname`, no git), `PlainDirectoryIsNoHub`.
  The worktree-subdirectory case needs a real hub and is `TestName_SweepsHubs`' cwd.
- tier 0, `TestName_Set`, through `runName` with a `fakeHubs` per hub (`fakeHubs` keys the five per-hub seams by board directory, records each `setOwners` request, and returns a set skip, pull error, owners, read error, write error or `OwnerWrite`; its `cwdHub` returns a set hub or none), rows:
  `InvalidNameIsRefusedAndWritesNothing`, `NoDirectoryIsRefusedNamingTheVariables`, `RelativeVariableIsRefused`, `UnreadableFileIsRefusedNamingItsPath` (a directory at the file's path),
  `RegistryReadFailureIsRefusedBeforeAnyWrite` (`hubs` errors: no file written, no `setOwners` call),
  `UnreadableBoardIsRefusedBeforeAnyWrite` (one hub's `readOwners` errors: refusal names that hub, no file written, no `setOwners` call on any hub),
  `CollisionIsRefusedNamingHubAndOwner` (no file written, no `setOwners` call; the way forward names the hand write and `lym adopt`),
  `OwnFormerNamePassesTheCollisionCheck` (`{"name":"BAD","names":["neve","osl46"]}` while a board records `osl46`; the output says the invalid name was dropped),
  `UndecodableFileLosesItsNamesAndSaysSo` (`not json`, host `box`: file `{"name":"osl46","names":["box","osl46"]}`, the output says the names were lost),
  `HandWrittenNameSweepsTheHostnameName` (`{"name":"osl46"}`, host `box`: file `names` `["box","osl46"]`, request `Replace` `["box"]`),
  `HostnameNamePassesTheCollisionCheck`, `PullSkipOrErrorIsCheckedLocally` (one hub's `pull` returns a skip, another's an error; both marked checked locally, both swept),
  `FailedHubIsReportedAndOthersSweep` (exit non-zero; a write error naming its own way forward reports it before the re-run), `LostEntryIsReportedPerEntry`, `UnpushedBeforeReportsPushed`, `NothingRecordedReportsNothingToSweep`, `EmptyEntryDropIsReported`,
  `CwdHubInTheRegistryIsSweptOnce`, `CwdHubOutsideTheRegistryIsSweptLast`;
  each successful row asserts the request: `Machine` the new name, `Replace` the former names, `Takeover` false, and that the output carries a non-empty `recoveries` list, never its text.
- tier 1, `TestName_SweepsHubs`, against real hubs; each subtest builds its own hubs, registry and file directory, so none starts from another's state.
  Setup per subtest: `boardkit.InstallReplay`; hub A from `hubforge.CopyHub`, its shortname rewritten with `fabricengine.WriteShortname` and recorded with `hubregistry.Record` in a registry under `t.TempDir()`; hub B from `hubforge.CopyHub`, not recorded, whose prime worktree's subdirectory is the cwd.
  The seed upserts beam projects through `boardengine.LoadConfig`, `boardengine.New` and `Board.UpsertTask`, then records their owners and the prime through `boardengine.SetOwners` with `Beam`, `Prime` and `Takeover`:
  A seeds a beam owned by `osl-1046`, a done beam owned by `osl-1046`, a beam owned by `far` and the prime owned by `osl-1046`; B's prime is `osl-1046`; a `hubforge.CloneSecond` of A pushes a beam owned by `osl-1046` and one owned by `neve` to origin only.
  `deps` uses the real hub seams, `liveHubs` over the temporary registry, a file path under `t.TempDir()` with no file, and `hostName` returning `osl-1046`.
  Each subtest asserts only what the tier-0 rows cannot: real Bolts, origin and the cwd resolution.
  Subtests: `CollisionWithAnOriginOnlyOwnerIsRefused` (`neve`, held only by origin before the verb ran: no file written);
  `RenameSweepsEveryHub` (`osl46`: origin holds `osl46` for both open `osl-1046` beams, the one only origin held before the verb ran included, and both primes, B's reached through the cwd subdirectory, and keeps the done beam's and `far`'s owners);
  `FailedHubIsSweptByTheReRun` (B's origin set to an unreachable URL with `gitkit.Git`: B is not swept and A is; the URL restored, the same command sweeps B and leaves the file's bytes unchanged).
  `TestSetOwners`' `ReplaceSetSelectsAfterThePull` (card 1) pins the selection after the write's own pull; these subtests pin the verb's sweep end to end.

**ImpactSummary:** confined to the new `internal/machinecli` package and its doc lines in `docs/overview.md`; it calls card 1's replace set and no existing caller changes.

**Commit:** 4: machinecli: set the machine name and sweep owners across a rename
