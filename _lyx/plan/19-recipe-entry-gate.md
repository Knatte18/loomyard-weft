# Card 19 — recipe-entry-gate

**Create:**
- `plan:internal/shedbuild#entryCheck` -> `type entryCheck struct`
- `plan:internal/shedbuild#parseEntryGate` -> `func parseEntryGate(row Row) ([]entryCheck, error)`
- `plan:internal/shedbuild#entryGateOf` -> `func entryGateOf(row Row, checks []entryCheck, env shedrecipe.Env) (shedengine.EntryGate, error)`
- `plan:internal/shedbuild#TestBuild_EntryGate` -> `func TestBuild_EntryGate(t *testing.T)`
- `plan:internal/configreg#CheckSettingKey` -> `func CheckSettingKey(module, key string) error`

**Edit:**
- `internal/shedbuild#Row`
- `internal/shedbuild#checkRowShape`
- `internal/shedbuild#Build`
- `internal/shedrecipe#Env`
- `internal/configreg#SettingLookup`
- `internal/loomcli#loomCLI.wire`
- `internal/battencli#battenCLI.wire`
- `internal/testkit/envkit#FullEnv`
- `internal/shedbuild#TestParse_ValueAndErrorCases`
- `internal/shedbuild#TestBuild_Errors`
- `internal/configreg#TestSettingLookup`
- `PATTERN.md`
- `pattern/PATTERN-recipe-setting-lookup.md`

**Extends:**
- `internal/shedbuild/doc.go`

**Intent:**
A recipe row may carry `entry_gate` beside its routing (D8, D9), parsed only in `shedbuild` (`PATTERN-sole-parsers`) and built into `ProducerDef.EntryGate`.

`Row.EntryGate` decodes `entry_gate` as one check or a list of checks, all of which must hold.
`parseEntryGate`, called from `checkRowShape` so `Parse` refuses a malformed check, reads each check as `<operand>`, `<operand> == <literal>` or `<operand> != <literal>`, with a YAML scalar literal; an operand is `module.key` or `budget.<Segment>`.
No or, no arithmetic, no free code: anything else, such as `a.b && c.d`, an empty check or an operand with no dot, is refused naming the row and the way forward of writing one of the three forms.

`Build` refuses, naming the row and the way forward, a `module.key` operand whose module or key the told `Env.SettingKnown` rejects, and a `budget.<Segment>` operand whose segment has no `Env.Segments` entry.
`Env.SettingKnown` is a new recipe-wide field, the known-key check `configreg.SettingLookup` makes, extracted as `configreg.CheckSettingKey` (an unregistered module or a key its template lacks, each with its way forward) and told by the callers, so `shedrecipe` and `shedbuild` import nothing new and batten's build needs no task worktree.
`loomCLI.wire`, `battenCLI.wire` and `envkit.FullEnv` set it.
A row with a `module.key` operand built against an `Env` whose `SettingKnown` or `Setting` is nil is refused by `Build` as a fail-loud wiring guard naming the row, the nil field and the way forward of filling it in the caller that builds the recipe, as `settingRef` refuses a nil `Env.Setting` today; no shipped caller reaches it.

`entryGateOf` returns the row's `EntryGate`, which reads at each evaluation: a `module.key` operand through `Env.Setting`, over the pair's config stack, a task's `run_settings` included; a `budget.<Segment>` operand as `Env.Segments[<Segment>].MaxBounces`, the segment's resolved soft cap.
A bare `module.key` operand reads a bool and holds when it is `true`; a bare `budget.<Segment>` operand holds when it is non-zero; a comparison compares the value with the literal.
A bare `module.key` holding any other type, such as the string `no` or the number `1`, is an evaluation error naming the row, the operand, the value and `lyx config <module>`, so the Command producer's refusal of a non-bool switch survives the move.
Its way forward is `lyx config <module> --resolved` in the task worktree, which names the layer holding the value, then fixing it there: with `lyx config <module>` from the prime for a hub or default layer, or with `lyx config <module> --layer local --set <key>=<bool>` in the task worktree for a value the local layer holds, since the board entry's `run_settings` reach the pair only at Seed-Child; then the recipe's resume command.
That command is `Env.Resume`, a new recipe-wide string beside `SettingKnown`: `lyx batten run <slug>` for batten, as the Command producer's switch refusals name it today, and `lyx loom resume` for loom and darn; `loomCLI.wire`, `battenCLI.wire` and `envkit.FullEnv` fill it.
The reason a false gate returns is the check's text with the values it read.
Bound: a gate skips only its own row; it reads only settings through the told lookup and resolved segment budgets (`PATTERN-recipe-setting-lookup`: no row gets an `Env` field).

`TestParse_ValueAndErrorCases` gains rows for a malformed check; `TestBuild_Errors` rows for an unknown module, an unknown key, an unknown segment and a nil `SettingKnown`; `TestSettingLookup` a row that the extracted check keeps the lookup's refusals.
`TestBuild_EntryGate` builds rows against an `Env` whose `Setting` is a fake and whose `SettingKnown` is `configreg.CheckSettingKey`, with no worktree: a true and a false bool, a comparison, a list of two checks, `budget.Code-Review` at 0 and above, and a `module.key` holding `no` and one holding `1`, each an evaluation error naming the told `Env.Resume`.
The shedbuild doc gains an Entry gate section, and `PATTERN-recipe-setting-lookup.md` names the entry gate's operands.

`PATTERN.md` gains this line under Shed and loom, after `PATTERN-command-row`:

```
- `PATTERN-row-entry-gate` — Leaving a recipe row out by config: its `entry_gate`, never a recipe rewrite or producer-side switch.
```

`PATTERN.md` already sits above its headroom line, so this card's `lint-pattern` step passes only when the overview is no longer than at HEAD.
In the same edit, these five entries take the shortened wording the parent's `changes` decisions of 2026-10-24 record, verbatim, which saves at least the new line's length; no rule changes:

```
- `PATTERN-command-row` — Adding side-action row to a recipe: a `Command` row over config keys, its command from template and board default only.
- `PATTERN-shed-recipe-registry` — Adding producer: one `map[string]Constructor` in `internal/shedrecipe` behind `Lookup`, bar `Implementer`'s.
- `PATTERN-shell-guard-table` — Guarding shell command: only in `internal/shellguard`, every engine rendering every row.
- `PATTERN-binary-change-reconcile` — Reconciling a hub's config after a binary change: only via `hubreconcile.Ensure`.
- `PATTERN-recipe-owns-failures` — A merge or test failure at Merge-In or Finalize: its fixer, a reroute or a halt to the orch.
```

**ImpactSummary:** Adds the `entry_gate` key and its build to every recipe and two recipe-wide Env fields; no shipped row carries it until cards 20 and 21.
