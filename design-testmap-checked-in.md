# Commit the coverage maps instead of keeping them in .lym

`testmap-checked-in` · Note · enhancement, gate, testkit, dobby

## Seen (operator, 2026-11-12)

The coverage maps are written to `<bolt>/.lym/testcover/<package>/map.json`, a directory the hub's `.lym/` exclude keeps out of git.
The first upkeep pass wrote 211 files, 56 MB, on 2026-11-12, started by the prime dobby because `test_map_upkeep` defaults to on; the operator had not authorised that build.
The operator wants the maps checked in, not kept in `.lym`.

## What a design has to settle

- Where they are committed: the code repository, or the Bolt records repository.
  Bolt is shared by the machines of a hub; the code repository is the one the maps describe.
- Size and churn: 56 MB of JSON, and a pass rewrites many files; every commit of them adds to git history.
  Whether the format should be smaller or more stable (sorted keys, one file per test, no timestamps in unchanged entries) before it is committed.
- Machines: a map carries an `EnvironmentStamp`, and a stamp that differs from the current one is a `MissEnvironment` that rebuilds the package.
  A map built on one machine and committed would be a miss on another, so the design says whether the stamp stays per machine, becomes shared, or the maps stay per machine in a different place.
- Who commits: the upkeep pass, the dobby, or a landing task; and what two machines' passes do when both commit.
- Whether `test_map_upkeep` is on by default for a hub, or the operator turns it on.

## Idea

A Motif that settles the points above before the maps are committed or the upkeep pass runs again on its own.
