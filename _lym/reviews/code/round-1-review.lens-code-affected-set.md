```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change, the set I found and whether the diff covers it:

1. **Removed `primeSection` / `noProjectSection` / `fileLink` names.**
   - Set: a repo-wide grep for these names and for `## Prime`, `No project` and `Prime section` found no consumer outside `render.go`, `doc.go`, `docs/overview.md` and `render_test.go`, all updated.
   - Covered.

2. **README layout (`## Prime` wrapper gone, project sections at `##`, root last).**
   - Set: `docs/overview.md:365`, `doc.go` README section, `render.go` header, `render_test.go` goldens.
   - The other board tests that read `README.md` do not assert the old layout: `replay_test.go:175-196` and `:587-596`, and `ownerset_integration_test.go:264-269`.
   - Covered.

3. **New output file `project-root.md`.**
   - Set: the Bolt replayer's `isBoardOverview` (`replay.go:66-73`) claims any `project-<slug>.md`, so it claims `project-root.md` too.
   - Set: the render manifest in `render.go:67-92` records the new file automatically.
   - Set: `replay_test.go:637-677` pin claims and overview names with `project-x.md`; they need no change.
   - Covered, apart from the collision in F1.

4. **`upLink` / `projectLink` replace `fileLink` and `fileHeader`'s root branch.**
   - Set: `fileHeader` and `projectSection` are the only callers, both changed.
   - Covered.

5. **PATTERN / `pattern/` background files.** None names the board render layout, and the commit carries no `Pattern-Departure:` trailer. Nothing to land, nothing violated.

**Findings**

**F1 — MEDIUM, design.** A real project with slug `root` overwrites `project-root.md`.
- Location: `internal/boardengine/render.go:103-111` and `:163`.
- `Render` stores the root's file under `projectFileName("root")` first. The loop over `ordered` then writes `projectFileName(twl.Slug)` for every project, so a project with slug `root` replaces the root's file. Its own README link (`projectLink`) then points at the same file as P0's link.
- The grep of `internal/boardengine` and `internal/boardcli` found no reserved-slug check. `validateSlugLength` (`task.go:357-362`) only caps length, and `NewTask` (`task.go:367-379`) checks only that the slug is non-empty and short enough. The replayer's prefix rule (`replay.go:70-72`) offers no protection.
- Evidence: the `docs/overview.md` / `doc.go` text says only "board project handles start at 1, so `P0` names no board entry". It claims slug safety for neither.
- Fix direction: refuse slug `root` for a project at write time, or give the root file a name no slug can take, and add a golden row.
- Slug `root` is plausible: `RootSubject = "root"` (`owners.go:14`) and `rootSlug` (`beamcli/target.go:15`) show the name is in common use.

**F2 — LOW, consistency.** The `doc.go:338` claim list names `README.md`, `project-<slug>.md` and `design-<slug>.md`, and it is accurate only because `root` reads as a slug. If F1 is fixed by a reserved name, this line must say so.

**Tool calls**
1. grep for removed names and `P0` / `project-root` across the repo, to find consumers outside `boardengine`. My context held only the `-l` file list from the earlier grep.
2. grep of other tests and docs asserting README or `project-*` output, to check whether they depend on the old layout. My context held only `render_test.go`.
