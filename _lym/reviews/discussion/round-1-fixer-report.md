# Fixer report — discussion round 1

Every finding's premise was checked against the code at HEAD, and every finding is fixed.

## Changes

- F1 (fixed as proposed): D6 admits the conversion only when the entry equals the arras project derived from the entry before it.
  The admitted change is: kind `project` and no status; the `sketchOf` field set cleared; title, labels, `project`, `beam` and the discussed flag unchanged; the body exactly the prefix, the part list and the former body; and `order` exactly the filed parts.
  A Bound line refuses any other scope change, and a D6 criterion covers a changed title or `depends_on`.
  The first D6 bullet now says "labels" rather than "module labels", to match.
- F2 (fixed differently): D6 keys the skip on a filing mark among the run's records under `shedrun.RunDir`, which Raddle-File writes when its board write lands with a project.
  It says the board's raddle store is not the key, and that the skip ends when the arras extension of `battencli.ClearMotifRun` clears the run's records.
  The D6 criterion now says the beam run's status writes reach the entry after `lym beam create`.
  Not made: the proposed example, "a mark in the run's raddle state under .lym".
  The raddle state is `loomengine.RaddleStatePath`, under the worktree's `_lym/raddle` (`internal/loomengine/raddlepaths.go:38,63`).
  `removeRunRecords` deletes only `shedrun.RunDir` and the scratch directory (`internal/battencli/cleanup.go:391,404`), so a mark in the raddle state would not be cleared.
- F3 (fixed as proposed, one claim narrowed): D2 states the split perch's `<perch>_auto_accept` key, which defaults to false and is a guard-category key.
  D4's Bound names the auto-accept route and bounds it by that default, the guard category, Raddle-File's checks and the draft status.
  A D2 criterion says that with the key off, a spent budget does not reach Raddle-File.
  Not made: the proposed word "operator-only".
  The guard category keeps a key out of the local layer only (`internal/configengine/doc.go:98`), not out of the worktree layer (`layers.go:40-46`).
- F4 (fixed differently): D10's rationale is corrected for the family outcome, and re-pointing is kept in both outcomes.
  Not made: leaving dependents on the converted project.
  `Store.Prune` removes each done member at landing (`internal/boardengine/store.go:836-880`).
  `metBelow` returns `len(members) > 0 && …` (`dependency.go:60`), so an open project with no member left never counts as met, and a dependent left on it would never start.
  The D10 criterion now says the dependent starts once the work has landed, and the ledger line gives the corrected reason.
- F5 (fixed as proposed): D8's first Bound bullet places the release in the verb's own path or a board method only the verb calls, never in `Board.SetStatus` or `Store.SetStatus`.
  The D8 criterion names batten cancel's claim clear.
- F6 (fixed as proposed): D4 says that in perch mode Raddle-Write ends on the finished proposal, the gate's pass stores the cold proposal, and a return moves the last proposal aside, as in auto mode.
  A D4 criterion covers it.
- F7 (fixed as proposed): after the filing, D7's refusal lists the filed slugs read from the run's filing.
  Its way forward is `lym loom decision add` on the part the decision concerns, once that part is released and its run has passed its record row.
  The D7 refusal criterion covers both ways forward.
- F8 (fixed as proposed, first option): D7's refusal is keyed on `perch` mode in the raddle state, which D4 records from the rows, so a `raddle` run keeps today's behaviour.
  A D7 criterion pins the raddle case.
- F9 (fixed as proposed): a board refusal of D10's re-point halts the run for its parent, naming the refusing entry, and never returns to Raddle-Write.
  A D10 criterion covers the halt.
- F10 (fixed differently): the third Open risks bullet is dropped.
  The matching Open ledger line is removed rather than changed to Resolved, because the ledger's first line already records that resolution: "Resolved (auto-pick): the fixer is a cold Burler fixer …".

## Self-check

- fix-proposal, BLOCKING (F4's proposal leaves dependents on a project that is never met once its members are pruned): fixed with my own change. D10 is restored to re-pointing in both outcomes, with a corrected rationale, a criterion that dependents start, and the ledger reason corrected.
- fix-claims, MEDIUM (D4's claim that a run's own worktree cannot turn the guard key on is false): fixed as proposed. The Bound now claims only the false default and the guard category.
- fix-consistency, LOW (support-log Turn 6 still says the cold-fixer question was raised under Open risks): fixed as proposed.
- fix-consistency, LOW (support-log Turn 6 still keys the skip on the raddle store): fixed as proposed.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
