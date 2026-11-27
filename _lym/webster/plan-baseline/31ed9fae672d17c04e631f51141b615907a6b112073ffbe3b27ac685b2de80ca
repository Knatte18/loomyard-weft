# Card 14 — merge-in-leftovers

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#Finalize.mergeParent`
- `internal/loomcli#landingDeps`
- `internal/testkit/envkit#nilLegal`

**Create:**
- `internal/lymdirs#IgnoreCheckDirName`
- `internal/shedrun#MergeInBaseFile`
- `internal/fabricengine#RemoveIgnoredLeftovers`
- `internal/landingshed#Finalize.removeLeftovers`
- `internal/landingshed_test#TestFinalize_RemovesIgnoredLeftoversOfADeletedDirectory`

**Extends:**
- `internal/lymdirs/doc.go`

**Uses:**
- `internal/landingshed/doc.go`
- `internal/loomcli/doc.go`
- `internal/fabricengine/doc.go`
- `internal/shedrun/doc.go`
- `internal/shedrun#ScratchDir`
- `internal/fabriclayout#HubScratchDir`
- `internal/landingshed_test#TestFinalize_OverRealHub`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D9 and issue #242: Finalize's merge-in removes the ignored leftovers of a directory the merge deleted, and goes on.

The base is the pre-merge task head, never the merge commit's first parent.
`Deps` gains `MergeInBasePath`, the file `shedrun.MergeInBaseFile` names under `shedrun.ScratchDir`, and `RemoveLeftovers func(base string) ([]string, error)`; an empty path or a nil seam removes nothing, so `envkit#nilLegal` lists the seam.
`mergeParent` writes the task head it read (`mergeBefore.taskHead`) to the base file before the merge-in, unless a base file already stands, whose base a resume keeps.
A resume takes the base from that file alone, never from an earlier merge's `task_head` nor a head read on the resume, so a pass killed between the merge-in and the merge record write still leaves its base, on a first merge-in and on a later one.
`removeLeftovers` runs after the merge-in, before the "after the merge-in" clean check,
and at `mergeParent`'s start, before the "before the merge-in" clean check, whenever the base file stands, so a resume past the merge reaches it; it removes the base file once the removal has run.
It logs each removed path; any other dirty or untracked path still halts with today's reason, and a tracked change is never touched.

`fabricengine.RemoveIgnoredLeftovers` does the git work, per PATTERN-fabric-boundary.
It lists each directory whose tracked files the merged tree deleted against `base`, and the untracked files under it, skipping a symlink and anything inside a nested repository.
It evaluates the ignore rules with git itself: it writes the base's per-directory ignore files of each such directory's path chain, read from the base's tree, into an empty scratch repository under the hub's never-tracked directory, `<HubScratchDir>/<lymdirs.IgnoreCheckDirName>`, with an empty `info/exclude`,
and runs `git -c core.excludesFile=<empty file there> check-ignore --no-index -z --stdin` with each candidate's path, so no user or global exclude file counts.
git-check-ignore(1) documents `--no-index` as checking the rules without the index, and gitignore(5) that a file under an excluded directory cannot be re-included.
It removes each file git reports ignored and returns their paths.
Bound: only files the base's rules ignore, under a directory the merge emptied of tracked files, are removed; every such file goes, an ignored local settings file or secret included, since the project is gone from the parent.
`landingDeps` wires the two fields, and `internal/lymdirs/doc.go` gains the line for `IgnoreCheckDirName` beside the other hub-level names.

`TestFinalize_RemovesIgnoredLeftoversOfADeletedDirectory`, tier1, red-first, over `TestFinalize_OverRealHub`'s hub fixture helpers.
Its rows, each where the merge base halts on the dirty tree: a parent commit that deletes `proj/` while the task worktree holds ignored `proj/bin/x` and `proj/obj/y` removes them and lands;
the same after a fast-forward merge-in; on a task's first merge-in with no earlier merge record;
on a resume after the merge with the base file standing; and on a resume after a kill between the merge-in and the merge record write, on a first merge-in and on a later one.
A row whose base holds a nested `proj/sub/.gitignore` with a negation `!keep.txt` under an ignored directory removes `keep.txt` too, pinning git's rule;
a non-ignored untracked file, a symlink and a nested repository under `proj/` stay, the first halting with today's reason; a tracked change still halts.

**ImpactSummary:** Adds one step before each clean check of the merge-in and two seams to Finalize's deps; a merge that deletes no directory is unchanged.
