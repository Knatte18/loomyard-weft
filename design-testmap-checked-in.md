# Commit the coverage maps instead of keeping them in .lym

`testmap-checked-in` · Note · enhancement, gate, testkit, dobby

## Seen (operator, 2026-11-12)

The coverage maps are written to `<bolt>/.lym/testcover/<package>/map.json`, a directory the hub's `.lym/` exclude keeps out of git.
The first upkeep pass wrote 211 files, 56 MB, on 2026-11-12, started by the prime dobby because `test_map_upkeep` defaults to on; the operator had not authorised that build.
The operator wants the maps checked in, not kept in `.lym`.

## Decided (operator, 2026-11-12)

Not in Bolt: Bolt is one branch shared by all branches, so a task's branch could not build its own maps there, and a branch should.
A map describes the code at a commit, so it follows the branch.

## What a design has to settle

- Not in the warp (operator, 2026-11-12): the maps are lym's, and only lym maintains them.
  They go in the weft, the records branch each pair has, which is per branch and lym's.
  Weft branches are never merged into each other (operator, 2026-11-12), so a task's maps do not travel to its parent by a merge.
  The design settles how the parent's maps move to the landed commit instead, for example by the parent's upkeep pass carrying them forward from the landed diff, as it does for any commit.
- Size and churn: 56 MB of JSON, and a pass rewrites many files; every commit of them adds to git history.
  Whether the format should be smaller or more stable (sorted keys, one file per test, no timestamps in unchanged entries) before it is committed.
- Machines: a map carries an `EnvironmentStamp`, and a stamp that differs from the current one is a `MissEnvironment` that rebuilds the package.
  A map built on one machine and committed would be a miss on another, so the design says whether the stamp stays per machine, becomes shared, or the maps stay per machine in a different place.
- Who commits: the upkeep pass, the dobby, or a landing task; and what two machines' passes do when both commit.
- Whether `test_map_upkeep` is on by default for a hub, or the operator turns it on.

## Idea

A Motif that settles the points above before the maps are committed or the upkeep pass runs again on its own.
