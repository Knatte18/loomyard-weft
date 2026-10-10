# Card 4 — own-json-flag-pin

**Create:**
- `plan:cmd/lyx#TestOwnJSONFlags` -> `func TestOwnJSONFlags(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `cmd/lyx#walkCommands`
- `cmd/lyx#newRoot`

**Intent:**
D3's guard: a test in `cmd/lyx/jsonhelp_test.go` walks the real tree `newRoot` builds through `walkCommands` and collects every command below the root whose own local or persistent flag set declares `json`, by command path without the `lyx` prefix, sorted.
It asserts the list equals the pinned set with exact equality, so a verb gaining its own `--json` later is a deliberate change reviewed against the classifier rule of Card 3.
The pinned set is what the walk reports today: the generic shed `status` verb under every recipe mount, which `lyx shed status --help`, `lyx loom status --help` and `lyx batten status --help` each list as a local `--json` flag; the implementer pins the walk's actual answer rather than the decision record's "shed status alone", and the test's doc comment says each pinned verb only reads.
The root's own persistent `--json` is the global flag and is excluded.
The test calls `newRoot` once and does not run commands, so it runs in parallel.

`cmd/lyx` sits at its row in `cmd/lyx/testdata/test-budget.yaml`, so the row rises by one in this card.

**ImpactSummary:** one test and one budget line; no production code changes.
