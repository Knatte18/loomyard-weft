```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fix carried out.

1. **BLOCKING**: `internal/sheddriver/tackler.go` (`settleFresh`) and `internal/shedarbiter/call.go` (arbiter refusal, through the new `sheddriver.ArbiterMaxSpawnsName()`).
   The F1 proposal, naming these in full, makes both texts name a possibly wrong key.
   They now read `shed.tackler_max_spawns is 0` and `off: shed.arbiter_max_spawns is 0`, but the resolved budget need not come from `shed.yaml`.
   - **Evidence:** `ResolveTackler`'s doc says "The spawn budget comes from a set override, 0 included, else shed.yaml's". `Override`'s says "a set one wins, 0 included".
   - **Evidence:** `internal/battenshed/template.yaml` holds its own `tackler_max_spawns` and `arbiter_max_spawns`.
   - **Concrete case:** `batten.tackler_max_spawns: 0` with `shed.tackler_max_spawns: 4` reports that the shed key is 0. It is not, and editing it changes nothing.
   - The old texts named no module, so they were not wrong. The updated `tackler_test.go` row now pins the shed name.
   - A fix needs the owning module carried into the settings, or a text naming both sources ("the run's module key, else `shed.…`").

2. **LOW**: `internal/configcli/configcli.go`, the reconcile help added for F2.
   "the reconcile that a clone, fabric reconcile, a start verb or a verb loading the loom or tapestry config runs" leaves out "outside `lym config`".
   - **Evidence:** the decision record's D5 says "The `lym config` verbs run no walk", yet `lym config loom` loads that config. The reviewer's wording ("every verb that loads…") has the same gap.
   - The edits to `preflightshed/doc.go`, `docs/overview.md` and `configengine.md` carry the qualifier. This one does not.

No finding on the other proposals:
- **F3 and F4:** each new test depends on the behaviour it names.
- **F6, F8 to F13:** no breaking case found.
- **Reworded `gate … exceeded gate.…timeout_min` detail:** `landingshed/verifygate.go:440` still matches, since it tests only the `gate build ` prefix. `verifytree/verify.go:517` cuts the log marker, which is unchanged.

Tool calls: one grep for parsers and quotes of the reworded texts (`Contains`/`Cut` sites, `.Detail` consumers, quotes in markdown), because my context did not show which code or docs consume those strings.
