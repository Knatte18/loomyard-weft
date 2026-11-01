```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, running each changed function on a concrete input.

**Per function, case run, verdict**
- `heading`: levels 2, 6 and 7 give `##`, `######` and `######`. Correct, and a heading never exceeds six `#`. Level 0 can't happen, because callers start at 1 or 2.
- `countedHeading`: level 6 with name `Tasks` and count 0 gives `###### Tasks (0)`. Correct. Every caller in `entrySections` and `projectSection` now passes an int, and the grep shows no stale string-level caller.
- `renderReadme`: Prime comes out at level 2, its sub-projects at 3, its own Tasks, Notes and Done at 3 and their subsections at 4. This matches the golden test's `### Tasks` and `#### Notes`. A board with no projects renders `## Prime` followed by the entry sections. Correct.
- `renderProjectFile`: the header is level 1, so sub-projects and own entries land at `##` and subsections at `###`. Correct. Facts and brief are not duplicated, since only `fileHeader` writes them.
- `projectSection`, root branch: `Task{}` gives the Prime heading and meaning line, then `projectBody(Task{}, level)`. Correct.
- `projectSection`, non-root branch: the member count covers open tasks, notes and sub-projects with `Project == slug`. That equals the sum of the project's Tasks, Notes and sub-project sections, and it excludes done entries. The result is zero for an empty project and the count agrees with the sections. A done member is excluded by `isDone`, so the count and Done never overlap. Correct.
- `projectBody`: for a chain of four nested projects, levels go 3, 4, 5, then 6. At the cap, both the child heading (level 7 → 6) and `entrySections` (7 and 8 → 6) clamp, so headings stay valid. A heading cannot collide with the project's own heading, because the sub-project sections come first and own entries last.
- `entrySections`: the level is now an int, and `sublevel = level+1` is clamped inside `countedHeading`. At level 6 the Tasks heading and its Ready subsection both come out as `######`. Valid markdown, though the hierarchy flattens, which is what the design asks for.
- `entrySections`, local `heading` variable: the notes loop declares `heading := otherNotesHeading`, which shadows the new package function `heading` inside that block. The block only calls `countedHeading`, so nothing breaks. It is a naming smell only.
- `childProjects`, unchanged: the root with a done parent still counts an open project under it as root-level. A nested open project under a project that is under a done project nests under its open parent, with no duplicate. Correct.
- Cycle or self-parent: a project whose `Project` is its own slug would recurse forever. The removed `projectSections` recursed the same way, so this is not a regression.
- Removed helpers: `columnEntry` is still used by the Done table, `emptySectionLine` by Ready, and the `priority` field by `noteTable`. Nothing is left dead.

**Findings:** none.

**Tool calls:** none. The inherited diff, full `render.go` read, test diff and grep already covered every definition and caller I needed.
