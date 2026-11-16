# Card 4 — machine-name-set

**Create:**
- `plan:internal/machinecli#setName` -> `func setName(d deps, cwd, name string, out io.Writer) int`
- `plan:internal/machinecli#nextFile` -> `func nextFile(current machinename.File, decoded bool, hostname, name string) fileUpdate`
- `plan:internal/machinecli#fileUpdate` -> `type fileUpdate struct`
- `plan:internal/machinecli#sweptHubs` -> `func sweptHubs(d deps, cwd string) ([]string, error)`
- `plan:internal/machinecli#checkCollision` -> `func checkCollision(boards []hubBoard, own []string, name string) error`
- `plan:internal/machinecli#hubBoard` -> `type hubBoard struct`
- `plan:internal/machinecli#sweepHub` -> `func sweepHub(d deps, hub, name string, former []string) hubSweep`
- `plan:internal/machinecli#hubSweep` -> `type hubSweep struct`
- `plan:internal/machinecli#recoveries` -> `func recoveries(name string, sweeps []hubSweep) []string`
- `plan:internal/machinecli#TestNextFile` -> `func TestNextFile(t *testing.T)`
- `plan:internal/machinecli#TestSetName` -> `func TestSetName(t *testing.T)`
- `plan:internal/machinecli#TestSetName_SweepsRealHubs` -> `func TestSetName_SweepsRealHubs(t *testing.T)`

**Edit:**
- `plan:internal/machinecli#Command`
- `plan:internal/machinecli#deps`
- `plan:internal/machinecli#productionDeps`
- `internal/machinecli/doc.go`

**Uses:**
- `internal/machinename#ReadFile`
- `internal/machinename#CheckName`
- `internal/machinename#File`
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#ReadOwners`
- `internal/hubregistry#Live`
- `internal/fabricengine#Bolt.Pull`
- `internal/fabricengine#Bolt.HasUnpushed`
- `internal/state#WriteJSON`
- `internal/lymcwd#Resolve`
- `internal/hubforge#CopyHub`
- `internal/hubforge#CloneSecond`
- `internal/output#ErrFields`

**Intent:** Make `lym machine name <name>` set the machine name: validate it, check it against every swept board, write the machine file keeping its names, and sweep this machine's owners on every hub (D5, D6, D8).

`Command`'s `name` leaf becomes `Use: "name [<name>]"` with `cobra.MaximumNArgs(1)`: no argument runs `showName`, one runs `setName`.

`deps` gains the hub fields, each filled in `productionDeps`:
- `registryHubs func() ([]string, error)`: the `HubPath` of every entry `hubregistry.Live` lists in `standalonestate.UserStateDir()`;
- `cwdHub func(cwd string) (hub string, ok bool)`: the `HubPath` of `lymcwd.Resolve(cwd)`, ok false when cwd lies in no hub;
- `pull func(hub string) (fabricengine.BoltSkip, string, error)`: `Bolt.Pull` of the hub's Bolt, with the skip options `boardengine.ApplySkipEnv` reads;
- `readOwners func(hub string) (boardengine.Owners, error)`: `boardengine.ReadOwners` of the hub's board;
- `hasUnpushed func(hub string) (bool, error)`: `Bolt.HasUnpushed` of the hub's Bolt;
- `setOwners func(hub string, req boardengine.OwnerRequest) (boardengine.OwnerWrite, error)`: `boardengine.SetOwners` of the hub's board.
A test hands in a file path and hubs under its temporary directory, so no test writes the developer's machine file or a real hub's Bolt (D8).

`setName` runs, in order, each step stopping on a refusal that writes nothing:
1. `CheckName(name)`; an invalid name is refused with the rule as its way forward.
2. `filePath`; with no config directory it refuses naming `XDG_CONFIG_HOME`, `HOME` and `LOCALAPPDATA`, and a relative variable is refused with `FilePath`'s error.
3. It reads the file through `machinename.ReadFile` at that path, never `machinename.Name`, so a bad file is repaired: a decoded file keeps its `names` and drops an invalid `name`; an undecodable one counts as no names and is reported as replaced.
4. `sweptHubs` lists `registryHubs` plus `cwdHub(cwd)` when ok, deduplicated and sorted; a registry read error is refused naming the record's path.
5. For each hub it calls `pull`; an error or a non-empty skip marks the hub `checked_locally_only` in the output, and the check reads the local board.
   It then calls `readOwners`; a read error is refused naming the hub, since the collision check cannot run without it.
6. This machine's names are the file's valid current name, every non-empty `names` entry, and `hostnameName()`'s name when it yields one; with no file, the hostname rule's name is the current name.
   `checkCollision` refuses a name that a board records as the prime owner or as any beam project's owner, done or not, when it is not one of those names, naming the hub and the owner.
   Its way forward: choose another name, or, when the name is this machine's own lost name, write `{"name": "<name>"}` to the file's path by hand.
7. `nextFile` computes the new file: `name` becomes the new name; the old valid name and the hostname rule's name are each appended to `names` when absent; every empty entry is dropped and reported; the new name is then moved to the end of `names`.
   It reports whether the result differs from the file on disk; the file is written through `state.WriteJSON` at the seam's path and lock only when it does, so a re-run with the name set writes only when `names` changes.
8. The replace set is every non-empty entry of the new `names` other than `name`. For each hub, `sweepHub` reads `hasUnpushed`, then calls `setOwners` with `Machine: name`, `ReplaceOwners` set and nothing else, and reports one of:
   - `failed`, with the error and the way forward `lym machine name <name>`, while the other hubs go on;
   - `lost`, each entry of `OwnerWrite.Lost` with its standing owner, the way forward `lym machine name <name>` for an owner that is a former name and `lym adopt <slug>` for one under another name;
   - `swept` with `Recorded`;
   - `pushed` when nothing was recorded and the Bolt was ahead of origin before the write and is not after it;
   - `nothing to sweep` otherwise.
9. The output carries the name, the former name, the new `names`, whether empty entries were dropped and whether an undecodable file was replaced, each hub's report, and the `recoveries` lines.
   It exits non-zero through `output.ErrFields` when any hub failed or lost an entry.

`recoveries` returns the recovery lines D6's window bound lists, naming each hub's own command where one applies:
a run paused as owned elsewhere: the dobby's probe lifts its own owner pause after the hub's sweep, else `lym tender resume <slug>`;
a hub whose sweep committed but did not push: re-run `lym machine name <name>`;
a Finalize halted by its owner check: `lym loom resume` in the run's pair;
a worktree skipped or refused as owned elsewhere: `lym beam start <slug>`, or `lym beam start root` for the prime;
and the advice to lift any pause with `lym tender resume` only after that hub's sweep reports done.

`doc.go` gains the set path: the collision check, the file write, the sweep, its bounds (it writes only this machine's file and owners recorded under its former names) and that it sweeps only the registry's hubs and the working directory's hub.

`TestNextFile` (tier0) is a table over `nextFile`: a hostname-named machine `osl-1046` with no file renamed `osl46` gives `names` `osl-1046, osl46`; a file with an invalid name keeps its non-empty names and adds the hostname name; an undecodable file gives the hostname name then the new name; a file with only `{"name": "osl46"}` adds the hostname name; an empty entry is dropped and reported; a re-run with the name set reports no change.

`TestSetName` (tier0) runs `RunCLI` with a temporary file path, a fixed hostname name and in-memory hub fakes for the hub fields, a hub's owners a map the fake `setOwners` rewrites by the replace set; it asserts the envelope, the file's bytes and each fake board's owners per row:
an invalid name and no config directory are refused with the file untouched;
a name a board records for another machine is refused naming the hub and the owner, with no board changed;
a name the file's `names` already holds passes though a board records it;
a hub whose `pull` fails is reported checked locally only;
a hub whose `setOwners` fails is reported failed with the re-run while the other hub is swept and the exit is non-zero, and a re-run with the same name then sweeps it;
a hub returning a lost entry is reported with its standing owner and way forward;
a hub ahead before the write and level after it is reported pushed;
and a rename's output carries the recovery lines and the lift advice.

`TestSetName_SweepsRealHubs` (tier1) builds two hubs through `hubforge.CopyHub`, hands them in through `registryHubs` and a temporary file path through `filePath`, and records beam projects with owners on their boards, one owner pushed from a `hubforge.CloneSecond` clone so only origin holds it before the verb's pull.
It asserts that renaming `osl-1046` to `osl46` rewrites, at each hub's origin, the prime and every open beam project owned by `osl-1046`, the origin-only one included, and leaves a done beam project and one owned by another name untouched;
and that `lym machine name neve` is refused, with the file unchanged, when only origin held `neve` as an owner before the pull.

**ImpactSummary:** Widens the one leaf the previous card created and adds the set path inside internal/machinecli; nothing outside the package calls it.
