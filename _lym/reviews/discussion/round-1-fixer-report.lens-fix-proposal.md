```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out (fix-proposal lens).

Proposals challenged: F1, F2, F3, F4, F5, F6, F7, F8, F9, F10, F11, F12, F13, F15, F16. F14 was folded into F1's rewrite, so it is not a proposal fixed as written.

1. **MEDIUM**, F1, D4 Rationale and D5's import, method and own-`Files` bullets.
   - **Summary:** Diffing from the map's `Commit` removes the hit-rate loss only for function-body changes. A map built after the worktree's base still misses on main's later non-body changes, seen in reverse.
   - **Case:** main lands a commit after the worktree's base that adds an import or a type to `internal/configreg`. testmap rebuilds B's map at that commit (D2 staleness). In the worktree, configreg's file lacks that import, so D4 sees the file as changed. D5 reads it as an import removed since `Commit`, which is a miss, so every importer of configreg runs whole.
   - **Second case:** B's own files changed on main after the worktree's base. D5's own-`Files` bullet then misses for B from any older base.
   - **Effect:** the D4 Rationale's "lets a map serve any tier base, including one that predates or postdates the build" overstates what the design delivers. The board's flow of a worktree whose base predates a landing still misses whenever that landing changed anything outside function bodies.
   - **Evidence:** the review's own count of 52 of 163 commits touching widely imported packages, plus D5 bullets 3, 5 and 6.
   - **Way forward:** reword the Rationale to the body-only claim, and add an Open risk or Bound for non-body landings since the base.

2. **MEDIUM**, F10, D5 fifth bullet: "An added or removed import, named, blank or dot, counts as such a change".
   - **Summary:** As proposed, an ordinary edit inside a function body becomes a whole-run miss for every importer whenever it adds or drops an import.
   - **Case:** a branch adds `fmt.Errorf` wrapping inside function F of `internal/state`, which adds the `fmt` import. `fmt` is already linked into every binary, so no `init` enters or leaves. D5 still runs every importer of state whole.
   - **Why the proposal is broader than its premise:** the premise is the `init` set changing. That happens only when the imported package enters or leaves the binary's link set. Combined with item 1, main-side import edits since the map's `Commit` widen too.
   - **Narrower sound rule:** a miss only for an added or removed import of a package that is not otherwise linked. For in-module packages, the map's covered set says whether a package is linked.

3. **LOW**, F4, D2: "testmap skips a package … when any covered package's directory holds a modified or untracked Go file".
   - **Summary:** One dirty Go file in a widely imported package in the prime stops testmap from writing any map for every package that links it.
   - **Evidence:** covered means every in-module package the binary links (D2's first paragraph). A dirty `internal/configreg` file in the prime therefore blocks almost every map build until it is committed.
   - **Alternative:** the review's own alternative avoids this. It records each covered package's hashes from the files the build read.

No finding on F2, F3, F5 to F9, F11 to F13, F15 or F16.
- F11 is safe because testmap runs each test in its own process (`internal/testcover/runner.go:2`), so `init` blocks count as covered by every test and a changed `init` selects every test of B.
- F7's `lym config gate` exists (`internal/configcli/doc.go:104`).

Calls made:
- I grepped testcover and testmappackage for how per-test coverage is gathered, to check whether `init` blocks are attributed per test for F11. My context did not have the runner.
- I grepped configcli for `lym config gate`, to check that F7's operator step names a real command. My context did not have configcli.
