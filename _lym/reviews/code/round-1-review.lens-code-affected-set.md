```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change, I checked what calls, reads, asserts or documents it outside the touched lines.

**Per change: set found, covered or not**
- **Reopen removal (`lym beam reopen`, reopen request, PR-Review reroute):** covered.
  - A grep outside tests and `_lym` finds no leftover verb, `WriteReopen`, `reopenRows`, `Reopen` seam or `answerReopen`.
  - The remaining hits are the retained path constants and their uses: `internal/hubgeom/dobbygeom.go:79`, `internal/pawlengine/landrequest.go:19,31`, `deps.go:89`, `retiredmove.go:71` and the `envkit` fixture.
  - These match the plan's shared decision on retained constants.
  - Mentions in `docs/research/**/calls.tsv` are historical measurement data.
- **Pawl rows (BeamDescribe producer removed, Parent-Align, Beam-Describe and PR-Review dropped from pawl):** covered.
  - No non-test Go source names `NameBeamDescribe` or the `BeamDescribe` constructor.
  - Remaining `NameParentAlign` uses (`internal/loomcli/start.go:165`, `internal/reportengine/wakes.go:104`, `internal/reportengine/timingspans.go:85`, `internal/loomrecipe/recipekeys.go:24`, `internal/loomcli/subjects.go:236`) serve the tapestry and darn recipes, which keep the row.
  - `internal/loomcli/doc.go:42` and `internal/loomrecipe/doc.go:47` carry the new row list.
  - `docs/overview.md:469` and the `docs/overview.md:463` verb list are updated and name no `reopen`.
- **Declared PATTERN changes:** covered.
  - `PATTERN.md` carries all four entries with the declared text, including the two shortened ones.
  - `pattern/` has no background file for `pull-request-mode` or `github-auth`, and `pattern/PATTERN-owner-machine.md` states the rule unchanged, so no background file is left wrong.
  - I did not check `pattern/PATTERN-heddle-instructions.md` for a leftover `(new)`.
- **Hold boundary (Parent-Align to Finalize):** covered.
  - `boardengine/runlock.go`, `boardengine/doc.go`, `battencli/beamfork.go` and the `orchcli` own-landing check are in the diff.
  - The `pawl-recipe.yaml` header comment is updated.
- **Orch role text and verb help:** one gap, below.
  - The new "Landing a beam" section in `contracts/orchrole/orch-role.md` is present and names no `reopen`.
  - The `ready`, `pr` and `describe` help texts were not in my context, so I did not check them against D14.
- **Commit messages and departures:** the `git log` bodies carry only `Co-Authored-By` lines and no `Pattern-Departure:` trailer, so no trailer needs judging.

**Findings**

1. **LOW, consistency:** the "Landing halts" section of `contracts/orchrole/orch-role.md` (lines 198-202) is stale for a pawl run.
   - Line 198 says "Parent-Align and Finalize fix their own merge and test failures … and no pull request waits for an approval".
   - Line 200 says "A parent that moved since Parent-Align".
   - Line 202 names a pawl run's escalated decision at Parent-Align.
   - Pawl no longer has a Parent-Align row.
   - A pawl Finalize now halts on GitHub refusals (pending check, dismissed review, closed pull request), with a way forward that names `lym beam pr status` and `lym loom resume`.
   - The new "Landing a beam" section (orch-role.md:62) covers only the halted-Finalize follow-up, not these halts.
   - The section was left as written for task runs, and D14 asked only for the landing workflow. A reader of the pawl halt path gets an outdated row name and a "no pull request waits" claim that no longer holds for a beam with a recorded or required pull request.
   - Evidence: `contracts/orchrole/orch-role.md:198-202`; the diff hunk for this file adds only lines 43-62.

No other affected-set member was left wrong in what I read.

**Tool calls (6 of 6)**
1. Grep for leftover reopen references outside `_lym` and tests. The manifest covered only diffs, not the whole tree.
2. Grep of `pattern/` background files for the changed entries' wording, plus the `pattern/` diff stat. Background files were not in the manifest.
3. Grep of `contracts`, `docs`, `README.md` and `cmd` for `Parent-Align`, `Beam-Describe` and `BeamDescribe`. This was a search for stale row names in docs.
4. Read `contracts/orchrole/orch-role.md` lines 190-215. The manifest held only the added hunk.
5. Grep of `docs/overview.md`, `contracts/stencils` and `contracts/specs` for beam landing descriptions (`beam ready`, `beam land`). Those files were outside the manifest.
6. Grep of Go sources under `internal` and `cmd` for `Beam-Describe`, `NameBeamDescribe`, `NameParentAlign` and `BeamDescribe`. This checked for stale row-name callers.
