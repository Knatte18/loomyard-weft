```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged each proposal the fixer carried out (F1–F5, F7–F15) by looking for a concrete case where the change is wrong.

**MEDIUM: D4 "new since those maps" (F4) empties the unreached list that the F1 reach turn and D5 rely on**
- Changed text: D4's new sentence "A changed function new since the base, whose package another package's map records as linked with file hashes that differ from the worktree's, is listed apart as new since those maps … never as unreached". Also the D3 reach turn condition "lists an unreached function with no caveat (D4)".
- Case: a darn adds `func parseX` with no test to a package whose map is listed as linked by any other package's map.
  - That darn's own edit changed the package's files, so every such linked-hash record differs from the worktree. The condition holds for every new function in every package another package links.
  - `parseX` is therefore listed as "new since those maps" and never as unreached.
  - The reach turn never fires for it. The D5 rubric also loses it as a "behavior the diff adds with no coverage" candidate.
- Result: the main case the operator asked for, a new untested function in a package other packages link, is hidden. Before the fix it was over-reported; now it is under-reported.
- Evidence: D3's reach-turn trigger, D5's "An unreached function is a candidate", and the D4 caveat text.

**MEDIUM: D2 matching by "package and name, whatever its file" (F9) is ambiguous for build-constrained variants**
- Changed text: D2's "A function is matched across the two sets by package and name, whatever its file".
- Case: a package holds the same function name in OS-specific production files. Both files count under "Only production Go files count", so the base and head each hold two functions with that key.
  - A change to the Windows variant can be compared against the Unix base text, which yields a false change or a missed one.
- Evidence:
  - `internal/beamcli/dobbyreopen_windows.go` and its Unix sibling both define `func reopenStdio`.
  - `internal/fabricengine/latewriter_windows.go` and its sibling both define `func isLateWriterError`.
- Possible fix: match by package, name and build constraint, or fall back to file when the name repeats.

**MEDIUM: the reach turn (F1) re-fires on every fix round for the same unreached set**
- Changed text: D3's "It does so at most once per HEAD tree", and the bound "one writer turn and one gate attempt per tree whose report lists an unreached function with no caveat".
- Case: a darn writer ends its reach turn unchanged because it judged the function needs no test, for example a function only a tier 2 test exercises indirectly with no spawn mark.
  - Every later commit makes a new tree: each Darn-Write rework and each Code-Burler-Fix round. Each new tree gets a fresh reach turn listing the same functions and costs one gate attempt.
  - A Code-Burler-Fix writer is fixing review findings, and the turn pushes it to add tests outside the review's scope.
- Keying the once-only record on the set of unreached functions rather than the tree would bound it to one turn per set.

**LOW: D3's linked-package dirty check (F3) has no source for the linked set of a package with no map**
- Changed text: D3's "neither it nor any in-module package its test binary links has a modified or untracked file".
- Case: a package the diff adds has no worktree map and no hub map. The record names nowhere the linked set comes from before the first measurement. The maps record linked packages (PATTERN-test-speed: "the hashes … of each linked package's files"), but only once a map exists.
- The check is therefore undefined for exactly the packages most likely to need a refresh.

**LOW: D1's file-set stamp (F8) can turn the index into a rebuild on every read, with no stated bound**
- Changed text: D1's "stamped with the set of map files in the directory … so an added or removed map file is a stamp difference".
- Case: the prime's `--map` run is ended by a moved HEAD after writing some maps. PATTERN-test-speed says "a moved HEAD ends the walk", and D1 has only that run write the index.
  - Every reader then rebuilds the 56 MB table in memory on every call until the next full map run. That covers each gate's reach step, every `lym gate reach` and every triage lookup.
  - D1's Bound sentence still names only the same-size rewrite case.

Calls beyond context:
- One Bash grep for same-named functions across `_windows.go` / Unix file pairs under `internal/`. My context had no listing of OS-specific files, and I needed it to build the F9 case.
