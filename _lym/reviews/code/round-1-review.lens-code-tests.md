```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per behaviour: pinning test**

| Behaviour | Pinned by |
|---|---|
| A beam orch sets `profile` on a new task whose `project` is its beam | `verbs_test.go` subtest "a beam's orch changes the entries of its own beam only", `upsert new` plus the `get` check for `profile == docs` |
| A beam orch sets `profile` on a new task under a group inside its beam (`BeamUnder` walks the group link) | Same subtest, `upsert deep` under `grp` |
| A beam orch changes `profile` on a stored member of its beam (stored link and payload link agree, so `links` has one element) | Same subtest, `upsert member light` |
| A new entry is not checked against a stored link (`stored.Slug != ""` guard) | Same subtest, `upsert new`. Without the guard `links` would gain `""`, which resolves to the prime and refuses the create. |
| An entry outside the beam is refused | Refusal row "an entry outside the beam" |
| A new entry under no project is refused | Refusal row "a new entry under no project" |
| A member moved out of the beam is refused (stored-link check) | Refusal row "a member moved out of the beam", payload link `""` against stored link `bm` |
| An entry moved into the beam is refused (payload-link check) | Refusal row "an entry moved into the beam" |
| The beam project itself is refused | Refusal row "the beam itself" |
| A non-orch role carrying the beam's slug is refused (`MatchesRole` branch) | Refusal row "another role in the beam" |
| The orch of another beam is refused (`beam != name.Slug`) | Refusal row "the orch of another beam" |
| The refusal message names the session and the way forward | The shared assertion in every refusal row |
| A slug-free strand, or none, is unrestricted | Pre-existing rows in `stepRunSettings`, unchanged by the diff |
| `BeamUnder` returns the project itself for an open beam | `parentbeam_test.go` "BeamUnderAnswersFromTheProjectLink", row `beam` |
| `BeamUnder` returns the beam for a group inside a beam | Same, row `group` |
| `BeamUnder` returns `""` for a done beam | Same, row `closed` |
| `BeamUnder` returns `""` for a non-beam project | Same, row `loose` |
| `BeamUnder` returns `""` for a project the board does not hold | Same, row `ghost` |
| `BeamUnder` returns `""` for an empty project | Same, row `""` |

**Findings (coverage gaps)**

1. LOW, class `design`, `internal/boardcli/runsettings.go:92-98` (`MatchesRole` branch), test `internal/boardcli/verbs_test.go:~820`.
   - A numbered orch role, such as `ly:bm:orch-2`, is admitted by `MatchesRole`, and no row pins it.
   - Breaking `MatchesRole` back to `==` would pass every row.
   - The fix is one row in the existing table.
   - Evidence: the only strands in the new subtest are `ly:bm:orch`, `ly:bm:loom` and `ly:other:orch`.

2. LOW, class `design`, `internal/boardcli/runsettings.go:99-102`.
   - A non-string `project` in the payload (`project, _ = value.(string)` collapses it to `""`) has no row.
   - The guard treats it as the prime link and refuses.
   - The only refusing `project` rows are `""` (moved out) and `"bm"` (moved in).
   - The store may reject a non-string `project` later, so this is low value and only matters if the guard's own outcome is meant to be pinned.

3. LOW, class `design`, `internal/boardengine/parentbeam.go:71-83`.
   - The `BeamUnder` error path has no test: `GetTask` failing, `ParentBeamOf` failing, or the `parentOf` seam returning an error.
   - The `BeamUnder` test builds the board without `Config.ParentOf`, so the seam branch of `ParentBeamOf`, the one the CLI sets (`cli.go:146`), is not exercised in the engine test.
   - Whether the boardcli subtest reaches that seam depends on `cliFixture`, which I did not read.
   - The row `group` (reached through `ParentBeamOf` with the seam nil) pins the non-seam path only.

4. Unparseable strand names now return nil, as `TaskSlug` did before.
   - No row pins it, and the behaviour is unchanged from the old guard, so I raise no finding.
   - A refusal of an unparseable name is a design choice the diff does not make.

**Test economy**

- The `BeamUnder` unit test and the boardcli subtest overlap only on the rows `beam` and `group`.
- The unit test uniquely covers `closed`, `loose`, `ghost` and `""`; the boardcli subtest uniquely covers the guard logic.
- This is not duplication, so I raise no finding.
- Both new tests are table-driven or share one subtest, so there are no per-behaviour redundant tests.

Tool calls: none; the diff, `runsettings.go`, `parentbeam.go` and `hubgeom/parentof.go` were already in context.
