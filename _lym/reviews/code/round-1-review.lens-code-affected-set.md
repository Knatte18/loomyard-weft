```yaml
lens: code-affected-set
outcome: reported
```

Per change, the affected set and whether the diff covers it:

- **Arras refusal** (`checkOfferedRecipe` in `validateWrite`). Callers are upsert, batch, merge, promote, set-status, set-deps, `MakeSketch`, `repointDependents` and the Bolt replay's `addEntry` and `changeEntry`. All of them meet the check. The store doc and the `boardengine/doc.go` paragraph are updated. The arras family-filing fixture (`startArrasRun`) is adapted. `boardcli/verbs_test.go:1077` asserts the profile defaults, not the refusal, so it is unaffected. **Covered.**
- **Arras visibility.** `lym board upsert --help`, the orch role, `ly:board` and `ly:board-policy` are updated. Error text and config comments built from the recipe constants are not. **Not covered** (F1). Glossary and overview text describing arras as a recipe a board task names is **not covered** (F3).
- **The Sketch line ignores `land_notice`.** `landnotice.go`'s file header and `rememberRuns` doc, `dobbyengine/doc.go:148` and the orch role are updated. The run-watch doc at `dobbyengine/doc.go:173` stays accurate. The config template comment at `internal/dobbyengine/template.yaml:6` is **not covered** (F2).
- **`RunView.Motif`.** Its one reader is `rememberRuns` and its one writer is `battencli.DobbyRuns`. Doc lines are updated and the test is renamed and extended. **Covered.**
- **Orch follow-up text.** It names `lym board promote` with `depends_on` and `lym board set-deps`, which match `cli.go:186-209` and `cli.go:1097`. The attach command it names is defined in `orch-role.md:482-483`. The raddle waiting line that triggers "split ready" is named nowhere in the orch role. **Partly covered** (F4).
- **Raddle stencils.** Every study item lands in the place the study names. The rubric's class list is extended. The new agent text addresses "the operator" in modes whose rendered `mode_rules` say no operator is present. **Not covered** (F5).
- **Commit messages.** No `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers, and none are needed. No assertion was loosened: `startArrasRun` only moves where the recipe gets set. **Covered.**
- **Declared PATTERN changes.** The directive declares none. **Nothing to land.**
- **Repository rules.** The refusal names its way forward and a test reaches it, as `PATTERN-refusal-way-forward` asks, although boardengine is not a bound module. The `doc.go` paragraph documents only its own package, so `PATTERN-code-conventions` holds. **Covered.**

Findings:

- **F1. MEDIUM, design.** Location: recipe-name messages and config comments built from the recipe constants.
  - Summary: error text and config comments still offer arras, though the board entry says no help text or recipe listing names it.
  - Evidence:
    - `internal/battencli/wire.go:616`: the unsupported-recipe error tells the user "a Board task's type must be … `arras` …".
    - `internal/shedrun/seed.go:109`: the unknown-recipe error lists `RecipeNames()`, which includes arras.
    - `internal/runprofile/template.yaml:2,4,26`: profile comments and defaults that the `lym config profiles` editor shows.
    - `internal/loomengine/template.yaml:52`: the split reviewer comment names "the arras recipe".
  - Fix: drop arras from these listings and comments, keeping the code that runs it.
- **F2. LOW, consistency.** Location: `internal/dobbyengine/template.yaml:6`.
  - Summary: the `land_notice` comment still says the setting alone decides whether dobby posts a line once a run lands. A landed motif run now gets a line whatever it reads.
  - Fix: add that a landed motif run's Sketch line goes regardless.
- **F3. LOW, consistency.** Location: `docs/reference/weaving-glossary.md:133` and `docs/overview.md:1706,1710,1439`.
  - Summary:
    - The glossary still calls arras "the recipe a board task names for a tapestry-sized note", which the board now refuses.
    - The overview lists arras among the recipes a Board task's `recipe` takes.
    - `overview.md:1439` says a Sketch "waits for triage", though the orch now follows it up.
  - Fix: state that arras is no longer offered and that motif serves a tapestry-sized note.
- **F4. LOW, consistency.** Location: `contracts/orchrole/role-autonomy.md:68` and the notice list at `contracts/orchrole/orch-role.md:84-99`.
  - Summary: "When the run waits for that approval" never names the line that says so: `raddle: the proposal for the Sketch &lt;slug&gt; passes its checks and waits for the operator's approval…`, from `internal/loomcli/raddleverbs.go:129`. The orch's list of what reaches it doesn't include it either, so the trigger for "split ready" is never stated.
  - Fix: name that line as the trigger.
- **F5. MEDIUM, consistency.** Location: `contracts/stencils/raddle/raddle-template-agent.md`, the new Step 3 and Step 4 lines.
  - Summary: the new rules address the operator in modes that have none. Those rules are:
    - "tell the operator which and why"
    - "tell the operator what to file by hand"
    - "say so to the operator"
    - "say which in the text around the table"
  - The same prompt's `{{.mode_rules}}` say "no operator is present" in auto and perch mode (`internal/loomengine/raddletable.go:63-74`). Only interactive mode has a pane and text around a table. An auto or perch agent gets a contradiction, and these notes reach no one.
  - Fix: say where those notes go in a mode with no operator, such as a proposal field or the parent.

Tool calls:
1. Grepped `land_notice` and arras in viewer files: the config comments and docs describing the landing notice were not in context.
2. Read `dobbyengine/doc.go:160-180` and searched `docs/overview.md` for notice and Sketch text: needed the run-watch doc and the overview passages the change touches.
3. Grepped the orch role for "attach" and the raddle waiting line: needed to confirm that "the run's attach command" exists and whether the waiting line is named.
