## Changes

All six findings are fixed. Tier0 and tier1 of `./internal/gatecli` pass; the final tier0 re-run after the self-check passed.
Commits: `9a29ba45` holds F1-F6 together, not one commit per finding, since the edits to `tiers.go`, `dotnetscope.go` and their tests interlock; a follow-up commit holds the self-check change to F3.

- **F1** fixed as proposed, with a reworded tail: `planTierRuns`'s explicit-targets refusal now ends with `roleClause`.
  The trailing only-the-prime sentence became "to add {targets} to a tier's command, run \"lym config gate\" from the prime", so the refusal still names the config step that `doc.go` promises.
  The `exact` row in `tiers_test.go` carries the new string.
- **F2** fixed as proposed: a `scopes_integration_test.go` subtest runs `--card` under dotnet with a tier0 command that has no `{targets}` and names another project.
  It asserts the unreached-project line on stdout and exit 0.
- **F3** fixed differently from the proposal: a `.dll` or `.exe` word names a specific target, so a command with only such words no longer reaches every project.
  Own change: an assembly whose name equals the project's file name without its extension reaches that project.
  The proposed change alone would warn for a command that runs exactly the selected project's assembly (fix-proposal fork, MEDIUM).
  Rows cover a mismatched assembly (`Fast.dll` against `A/A.csproj`) and a matching one (`A/A.dll`).
- **F4** fixed as proposed: `isSameProjectFile` compares a bare file name by base name and a word with a directory part by path, equal or ending at a directory boundary.
  A row covers `A/Tests.csproj` against `B/Tests.csproj`, and one covers a leading `./`.
- **F5** fixed as proposed: `printRunWholeNotices` prints the run-whole notices before each successful return of `planTierRuns`; the text is unchanged.
  The `planTierRuns` doc comment says the line prints only once no refusal can follow.
- **F6** fixed as proposed: rows for a run with no invocation, a project selected by two tier shares, and a backslash project path.
  `doc.go` states the reach rule with the assembly case.

## Gate repair

The comment gate flagged two lines of this round, so no test ran.
`doc.go`'s reach sentence passed 120 characters and became two sentences, one per line.
The `isSameProjectFile` doc comment wrapped mid-sentence and now holds one sentence per line.
A third commit holds the repair; the tier0 gate passes after it.

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Self-check

All four forks returned a report; each is saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency, LOW, the no-test-class and `-skip` notices still print before a later refusal: no change.
  The review named only the run-whole notice, and those two lines were not moved by this change, so the rubric does not count it as a finding.
- fix-consistency, LOW, the `planTierRuns` doc comment silent on the deferred notice: fixed as proposed.
- fix-proposal, MEDIUM, a `.dll` named after the selected project warned falsely: fixed with my own change, as under F3.
- fix-proposal, LOW, the `roleClause` "these steps" reads as covering the drop-targets way forward: no change.
  `doc.go` requires every such refusal to close with that clause, and the reviewer proposed it.
- fix-proposal, LOW, directory-boundary suffix matching can conflate `A/Tests/Tests.csproj` with `B/Tests/Tests.csproj`: no change.
  The cost is a silent warning in a rare case, against false warnings when a command word and a project path are relative to different directories.
- fix-affected-set, LOW, same as the consistency lens's notice finding: no change, for the same reason.
- fix-affected-set, F1 and F3/F4: no findings.
