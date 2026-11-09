# One new loom config key fails dozens of tests that pin the whole struct

`config-struct-test-per-key` · Task · bug, loom, config, testkit · running · Worktree-Create

## Seen (orch, 2026-11-09)

Adding loom config keys in `arras-recipe-perch` failed 56 tests in the first tier 0 run, 47 of them cases of `TestLoadConfig_Loads` in `internal/loomengine`.
Each case compares the whole loaded config struct, so one new key breaks every case; the rest were `TestLookup` and `TestModules_EveryTemplateKeyClassified` in `internal/configreg`, `TestWire_ReviewKeysReachTheEnv` in `internal/loomcli` and `TestRecipes_KeySet` in `internal/shedcli`, which pin the key set.
The fix of those tests, and the gate attempt it needs, cost the run most of an hour.

## Idea

Make each case of `TestLoadConfig_Loads` assert only the fields it is about, so a new key leaves the table alone.
Keep one test that pins the whole key set, since the classification test is meant to fail for a key nobody classified, and make its failure message say which key to add where.
Check the other tests that compare a whole struct or a whole key list, and list those that fail for any added key.
