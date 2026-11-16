```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. I traced every guard, refusal, branch and test row that cards 1, 3 and 4 add to a real entry point. Card 2 adds no branch.

**Per guard, reachable or not**

Card 1 (`writeOwners` / `SetOwners` with `ReplaceOwners`):
- An empty string in the set matches nothing: reachable. The `ReplaceSetRewrites…` row passes `""`, and `beamer` is unowned, so the guard fires.
- A done entry is not selected: reachable through `closed`.
- An owner outside the set is kept: reachable through `kept`.
- Task and grouping entries are not selected: cannot match, since `ownersOf` reads an owner only on a beam project (`owners.go:73-88`). This holds; it is not a defect.
- An owner only origin holds is selected after the pull: reachable through `ReplaceSetSweepsAnOwnerOnlyOriginHolds`.
- A selection that writes nothing commits nothing: reachable through `ReplaceSetWithNoMatchCommitsNothing`.
- An entry both `Slugs` and the set name is written once: **no row reaches it**, and card 4 never sets `Slugs` (finding 7).
- A replace-selected entry reported in `Lost` when a competing origin write keeps it: reachable through `BeforePush` + `pushFromOther`, but **no row reaches it** (finding 6).
- The commit subject with `replacing …`: the shape card 4 sends (no `Slugs`, no `Prime`) is **unspecified and has no row** (finding 8).

Card 3 (`showName`):
- Valid file, no file, no config directory, undecodable file, invalid name and relative variable: all reachable through the `filePath` seam. `TestShowName` names a row for each.
- No file while `hostnameName` errors: **unprescribed** (finding 9).

Card 4 (`setName`):
- Step 1, `CheckName` refusal: reachable. `cobra.MaximumNArgs(1)` lets any single string through, and a `TestSetName` row covers it.
- Step 2, no config directory: reachable, with a row. Its upstream `CheckName` passes a valid name, so nothing rejects the input first.
- Step 2, relative variable: reachable through the seam, but **no `TestSetName` row** (finding 1).
- Step 3, undecodable file replaced and invalid `name` dropped: reachable, but **reached only by `TestNextFile`**, which bypasses the `setName` read and its output (finding 2).
- Step 3, file that does not read: an I/O error from `ReadFile` returns `exists=false` with an error (`machinename.go:60-61`). The card **leaves this branch unprescribed** (finding 3).
- Step 4, registry read error refusal: reachable through the `registryHubs` fake, but **no row** (finding 1).
- Step 4, the hub of the working directory: `cwdHub` uses `lymcwd.Resolve`, whose strict anchor gate drops a cwd that lies in a hub but off its anchor. The guarded site **recurs unguarded** (finding 5).
- Step 5, `pull` error or skip marks the hub checked locally only: reachable, with a row covering the error half.
- Step 5, `readOwners` error refusal: reachable with a corrupt `board.json`, but **no row** (finding 1).
- Step 6, collision refusal: reachable, with a fake row and the real-origin row in `TestSetName_SweepsRealHubs`. `CheckName` passes `neve`, so no upstream check rejects it.
- Step 6, a name `names` already holds passes the collision check: reachable, with a row.
- Step 7, no write when nothing changed: reachable on the row that re-runs after a failed hub.
- Step 8, `failed`, `lost` and `swept`: reachable, with rows.
- Step 8, `pushed`: reachable in production, because `PushRecorded` pushes the branch whatever `rec` holds (`bolt.go:141-147`), so an earlier unpushed commit goes out. The fake row's "level after" state needs a second `hasUnpushed` read that the card's `sweepHub` never makes (finding 4).
- Step 8, `nothing to sweep`: reachable.
- Step 9, non-zero exit on a failed or lost hub: reachable, with rows.
- The D8 bound that tests sweep only temporary hubs: **the site recurs** in `TestSetName_SweepsRealHubs` (finding 5).

**Findings**

1. MEDIUM, consistency, card 4 `TestSetName`: three refusals that `setName` prescribes have no `TestSetName` row.
   - Step 2: the relative variable. `TestShowName` covers only the show path.
   - Step 4: the registry read error.
   - Step 5: the `readOwners` error.
   Each is reachable through its fake seam (`filePath` returning an error, `registryHubs` returning an error, `readOwners` returning an error). Evidence: `_lym/plan/04-machine-name-set.md:54,56,58` against the row list at `:85-92`.

2. MEDIUM, consistency, card 4 `TestSetName`: no `setName` row reaches the repair branches. These are step 3's undecodable-file replacement and invalid-`name` drop, and step 9's output fields for a replaced file and dropped empty entries. Only `TestNextFile` (`:82`) covers them, and it calls `nextFile` directly. Two acceptance criteria are therefore unreached at the verb:
   - empty entries dropped "and says so in its output";
   - a hand-written `{"name"}` file whose owners are swept under the hostname rule's name.
   Evidence: `04-machine-name-set.md:55,70,82,84-92`.

3. MEDIUM, design, card 4 step 3: a machine file that exists but does not read (a permission error, or a directory at the path) is left unprescribed. `ReadFile` returns `(File{}, false, err)` here (`internal/machinename/machinename.go:60-61`), which is neither "no file" nor `decoded=false`. The card does not say whether to refuse or replace, and no row reaches it. The show path (card 3, `03-machine-name-show.md:43`) does prescribe this case.

4. LOW, consistency, card 4 `sweepHub`: `pushed` requires the Bolt to be ahead before the write "and not after it". The card's `sweepHub` reads `hasUnpushed` only before calling `setOwners` (`04-machine-name-set.md:64,68`), yet the row "ahead before the write and level after it" (`:91`) needs a second read.

5. MEDIUM, design, card 4 `cwdHub` and `TestSetName_SweepsRealHubs`: two problems at the same site.
   - `cwdHub` maps `lymcwd.Resolve(cwd)` to "ok false" whenever cwd lies in no hub. `Resolve` refuses any cwd other than the exact anchor (`internal/lymcwd/lymcwd.go:76-84`), so a run from a worktree subdirectory or the hub container silently skips that hub. D6 says the verb sweeps "the hub of the working directory when it lies in one". The signature has no error, so other `Resolve` errors are swallowed too.
   - `TestSetName_SweepsRealHubs` (`04-machine-name-set.md:94`) overrides only `registryHubs` and `filePath`. It leaves `cwdHub` and `hostnameName` at production, so the test reads the process directory and the developer's real hostname:
     - the hostname joins `names` and the replace set, so the outcome depends on the host;
     - only `Resolve`'s anchor gate keeps the developer's own hub off the sweep list, which D8 forbids relying on (`decision-record.md:102`).
   RunCLI's empty cwd reads the process directory (`internal/hibernatecli/cli.go:182-186`).

6. LOW, consistency, card 1 `TestSetOwners`: no row reaches a replace-selected entry landing in `Lost` after a competing origin write. D6's `lost` report rests on that path, and card 4 only fakes it. The path is reachable through `BeforePush` + `pushFromOther`, as `CompetingWriteLosesToOrigins` does for a named slug (`ownerset_integration_test.go:453-470`). Evidence: `01-owner-replace-set.md:25,31-35`.

7. LOW, design, card 1: "an entry both name is written once" (`01-owner-replace-set.md:24`) guards a state no caller produces. Card 4 sends `ReplaceOwners` and nothing else (`04-machine-name-set.md:64`), and no row reaches it.

8. LOW, consistency, card 1 commit subject: the only subject shape the real caller produces is unspecified and has no row. Card 4 sends no `Slugs` and no `Prime`, so `subjects` is empty (`ownerset.go:101-105`). The card does not say whether the subject reads `board: owner neve for , replacing …` or something else. With `""` in the set, the joined list also carries a trailing empty item. The example `for prime, replacing box` (`01-owner-replace-set.md:26`) needs `Prime: true`, which no planned caller sets.

9. LOW, consistency, card 3 `showName`: with no file, or no config directory, and `hostnameName()` returning an error, the output is unprescribed and no row reaches it (`03-machine-name-show.md:42`).

Tool calls:
- `internal/hibernatecli/cli.go` lines 170-200 and `fabricengine.Bolt.PushRecorded`: to check how RunCLI handles an empty cwd and whether a no-commit owner write still pushes. Neither was in my inherited context.
