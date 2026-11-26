```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix change set.

**Findings left unchecked:** none.
F1 to F4 were searched.
F5 to F7 were checked from context: the new rows cover every `sourceText` branch, the root and one beam project file, the done-beam and non-beam filter, and the failed read.

**Findings**

1. **LOW**: F1, `internal/boardengine/doc.go`, README section.
   - The text still says "The section and file header of the root and of each open beam carry a mode line".
   - After the fix, an open beam with no worktree on this machine has none.
   - Only "A failed read renders the board with no mode line" follows, and it describes the whole read.
   - The fix updated `BeamModes.Beams`, `hubgeom/doc.go` and the summary's Open risks, but not this sentence.
   - The same unqualified sentence is in `_lym/landing/summary.md` under "What the board files show".

2. **LOW**: F1, the legend in `internal/boardengine/beammodes.go#modeHeader` ("A setting that the root and every beam take from the hub-wide default is stated here alone").
   - `everyPlaceHubWide` ranges over `b.modes.Beams`, which no longer holds an absent beam.
   - Suppose every present place takes autonomy from the hub-wide default, and the absent beam sets its own on its owner machine.
   - The label is then dropped from every line, and the legend's "every beam" tells the reader the absent beam takes the default too.
   - `doc.go` has the matching sentence.

3. **LOW**: F1 and F5 to F7, the Tests section of `_lym/landing/summary.md`.
   - It was not updated for the new coverage: the absent beam left out in `TestBeamModes`, the two inherited-source rows, the project-file lines, the open-beam filter and the failed read.

**Checked and complete**

- **F2:** the only `Beam PR` sites are the legend and the docs the fix updated.
- **F3:** `primeOfBoard` no longer calls `dotgit`.
- **F4:** `config.go` and `dobbyengine/doc.go` are the only two lists of the per-task readers the search found, and both name `LookupWithSource`.
- **`TaskSettingChain` docs:** `hubgeom/tasksetting.go:20` and `dobbyengine/tasksetting.go:23` still correctly describe what a run does.

**Outside scope:** `internal/hubgeom/autonomy.go:62` (`resolvePrimeWithoutGit`) also calls `dotgit.ListWorktrees` directly. The fix did not touch or reach it, so it is not counted as a finding.

**Tool calls**

1. `git grep` for absent-beam and mode-line wording: my context lacked the repo-wide sites. It failed on pathspec syntax.
2. `git grep` for per-task reader lists: my context lacked the repo-wide lists. It failed the same way.
3. `git grep` for `dotgit.ListWorktrees` and `Beam PR`: my context lacked the other callers. The first search answered and the second failed the same way.
4. Combined `git grep` rerun for the F1, F2 and F4 sets, plus a span of `autonomy.go`: needed because of those failures, and my context did not hold that file.
