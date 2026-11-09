```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. For each change the diff makes, I looked for everything outside the touched lines that reads, calls or asserts it.

**Per change, the set found:**
- **`RaddleMode` moved from `loomshed` to `loomengine`, and `RaddleModePerch` added.**
  - A grep of `internal` finds no remaining `loomshed.RaddleMode*` reference.
  - The callers I found are `raddlestate.go`, `raddlewrite.go`, `raddlefile.go`, `entries_raddle.go`, `envkit.go`, the `shedrecipe` fixture and the loomshed, loomengine and loomcli tests. All are updated: covered.
  - The `loomshed` import allowlist already holds `loomengine` (`internal/loomshed/seam_enforcement_test.go:31`). The `loomrecipe` and `shedrecipe` allowlists gain it in this diff: covered.
- **`RaddleTable` takes a mode, not a bool.** The wiring, the `shedrecipe` `Table` closure, the `envkit` and fixture stubs and `raddletable_test.go` are updated: covered.
- **`RaddleWriteDeps.SplitPerch` seam.** I found three places that need it:
  - the `raddleWriteEnv` wiring;
  - the nil-seam check list in `entries_raddle.go`;
  - the test fixtures: `envkit`, the `shedrecipe` fixture and the battencli `intentionallyNil` map.
  
  All three are updated: covered.
- **`raddleBoardContext` sets `Perch`.** It reaches the gate, both verbs and Raddle-File's checks through the one context builder. `RecheckBoard` needs no perch option, and the summary says why: covered.
- **`freshRaddleEntry` goto branch.** The table test is rebuilt. I found no other caller or fixture that builds the old goto shape: covered.
- **Stencil opening line.** The edit is minimal and drops no rule. The mode text arrives through `{{.mode_rules}}`, and `raddleModeRules` names both outcomes. `raddle-proposal-spec.md` gains the perch section, the `perch-outcome` check and the `slug-sketch` exemption: covered.
- **`decision add` refusal.** Both refusal rows have tests, as does the non-perch row. The package doc is updated. The refusal text has no denied recovery step, and the way forward names `lym loom goto`, `resume` and `decision add`: covered.
- **Commit messages.** The range has no `Pattern-Departure:` trailer, and the diff needs none. `PATTERN.md` and `pattern/` are untouched, and the directive declares no PATTERN changes: nothing to land.

**Findings:**

1. **LOW, consistency.** `internal/loomengine/template.yaml:5`
   - The shipped comment on `raddle_auto_approve` says false "has the agent put the proposal to the operator in its pane" and that the key is "read once, at Raddle-Write's first spawn in a run".
   - The diff changes the key's meaning: a split-perch run never reads it. It updated the `config.go` doc and `ConfigSpec` comments but not this one.
   - Evidence: `internal/loomengine/config.go:407` carries the new wording and `template.yaml:5` does not.
   - Fix: append "; never read in a run whose recipe rows hold a split perch".
   - This is a doc-only change. No meaning-change reinterpretation under `PATTERN-config-layers` is needed, because the key's value semantics are unchanged.

2. **LOW, consistency.** `contracts/stencils/raddle/raddle-template-agent.md:14`
   - The heading still reads "split a Sketch into the entries the operator approves", while line 21 now says the operator or the split perch approves.
   - Fix: reword the heading to "the entries that get approved", or drop "the operator".

3. **NIT, consistency.** `contracts/recipes/raddle-recipe.yaml:3`
   - The header comment lists only the operator's approval and auto mode as the two session ends. It does not mention perch mode.
   - Fix: none is strictly needed, because this recipe is never in perch mode. Add a clause only if you want the comment to stay complete.

**Tool calls:**
1. Grepped md, yaml and go files outside tests for `raddle_auto_approve`, mode names and "interactive mode" text, plus the stencil's approval lines. My context lacked docs and non-diff callers.
2. Checked the `loomshed` import allowlist and the `loomengine` mentions in `PATTERN.md` and `pattern/`. My context lacked the allowlist file because the diff did not touch it.
