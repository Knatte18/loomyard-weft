---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/decision.go:904
    summary: perchFilingRefusal's row guard and its read-error refusal have no test row, so dropping the guard would refuse every perch-mode decision add unnoticed
    fix: In internal/loomcli/decision_test.go add an AppendsAndCommits row with the fake's raddlePerch set and the run at a row other than Raddle-File, and a row for a recipe with no RaddleFile row, each asserting the decision is appended. Add TestDecisionVerb_Refusals rows where the fake's raddleFileRow, raddlePerch and raddleFiled closures each return an error, asserting the refusal opens "read the run's raddle filing" and carries its way forward clause.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/raddlewiring.go:76
    summary: raddleRunFiling has no test, though the post-filing refusal's slug list and its created-stamp match rest on it
    fix: In internal/loomcli add a table test of raddleRunFiling over a temp hub board and raddle store, with rows for no store entry, a store entry whose Sketch differs, a store entry whose Created differs from the board entry's, a matching entry returning its Filed slugs, a matching entry whose board task is gone (still returns Filed), and a board read failure.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/raddlecontext.go:35
    summary: raddleBoardContext setting Perch from the run's raddle state is untested, and it is the one signal that applies the D5 outcome checks in the gate, both verbs and Raddle-File
    fix: In internal/loomcli/raddlecontext_integration_test.go add a case that writes a raddle state with mode perch and asserts the returned BoardContext has Perch true, and a case with no state file asserting Perch false. Add a case for an unreadable state asserting the wrapped "read the raddle run's state" error.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/raddlewiring.go:157
    summary: the StoreCold closure's perch-mode store and interactive-mode skip have no test
    fix: In internal/loomcli add a test that builds raddleWriteEnv over a temp worktree and calls Deps.StoreCold with the state in perch mode and then in interactive mode, asserting the cold proposal is stored only in the first.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/loomshed/raddlewrite.go:137
    summary: the firstMode path where SplitPerch returns an error has no test
    fix: In internal/loomshed/raddlewrite_test.go add a TestRaddleWrite row where the fixture's SplitPerch returns an error, asserting Call returns the error naming the split-perch read and that the state records no mode and the inner producer did not run.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomengine/template.yaml:5
    summary: the shipped comment on raddle_auto_approve still says the key is read at every run's first spawn, though a split-perch run never reads it
    fix: In internal/loomengine/template.yaml extend the comment on raddle_auto_approve with "; never read in a run whose recipe rows hold a split perch", matching the wording the diff added to config.go.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/raddle/raddle-template-agent.md:14
    summary: the stencil heading still says the operator approves the entries, while the opening line now says the operator or the split perch approves
    fix: In contracts/stencils/raddle/raddle-template-agent.md reword the heading at line 14 so it no longer names the operator alone as the approver, for example "the entries that get approved".
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/loomrecipe/recipekeys.go:158
    summary: RowsHoldSplitPerch compares the artifact path without cleaning it, so a row naming ./_lym/raddle/proposal.yaml is not read as a split perch
    fix: In RowsHoldSplitPerch compare filepath.Clean(filepath.FromSlash(text)) to the proposal path instead of filepath.FromSlash(text).
---

### [MEDIUM] perchFilingRefusal's guard and error paths are untested

**Location:** internal/loomcli/decision.go:904
**Issue:** The two refusal rows and the non-perch Raddle-File row pin the happy refusals. No row shows a perch-mode run at another row, or a recipe with no RaddleFile row, still appends. The acceptance criterion confines the refusal to Raddle-File and later rows. The read-error refusal is unreached too, which the repository's refusal rule asks a test to reach.

### [MEDIUM] raddleRunFiling is untested

**Location:** internal/loomcli/raddlewiring.go:76
**Issue:** The decision verb's tests replace it with a fake closure. Its store read, the Sketch check, the created-stamp match and the gone-from-board case, which one-task filings reach, are never run. The post-filing refusal names the slugs it returns.

### [MEDIUM] raddleBoardContext's Perch is untested

**Location:** internal/loomcli/raddlecontext.go:35
**Issue:** D5's acceptance criterion needs the checks to apply in perch mode in the gate, both verbs and Raddle-File. All of them take `Perch` from this builder, and no test reads it for a perch-mode state.

### [LOW] StoreCold's perch-mode behaviour is untested

**Location:** internal/loomcli/raddlewiring.go:157
**Issue:** The closure changed to store in every mode but interactive. No test names perch mode against it.

### [LOW] firstMode's SplitPerch error path is untested

**Location:** internal/loomshed/raddlewrite.go:137
**Issue:** The new failure path has its own message, and no row makes the seam fail.

### [LOW] template.yaml comment on raddle_auto_approve is stale

**Location:** internal/loomengine/template.yaml:5
**Issue:** The diff updated the key's meaning in `config.go` and `ConfigSpec` but not in the shipped template comment, which still says it is read once at each run's first spawn.

### [LOW] Stencil heading names only the operator as approver

**Location:** contracts/stencils/raddle/raddle-template-agent.md:14
**Issue:** Line 21 now names the operator or the run's split perch, and the heading above it still says "the operator approves".

### [NIT] RowsHoldSplitPerch does not clean the artifact path

**Location:** internal/loomrecipe/recipekeys.go:158
**Issue:** A `./`-prefixed path to the proposal fails the match, and the run falls back to auto or interactive mode. The fallback is the safer direction.

## Rejected

- A NIT from code-affected-set that `contracts/recipes/raddle-recipe.yaml:3` does not mention perch mode: the recipe is never in perch mode, so its header stays true.
- Handler check of the decision-add refusal against D7's "from the Raddle-File row on": the refusal fires at the RaddleFile row, done or not, and the recipe ends there, so no later row exists to refuse at; not a defect.
- Handler check of D4, D5 and D7 against the diff: `freshRaddleEntry` matches what `Goto` and `Relaunch` write, `raddle_auto_approve` is unread in perch mode and a `raddle` run keeps its interactive approval, `raddle-approve` refuses naming the perch with a way forward, the checks refuse beam, multi-part tasks and a foreign project slug, and the exemption covers only the run's own project slug. No scope gap or excess found beyond the notice-free `raddle-propose` in perch mode, which follows from the mode having no operator.
- No `Pattern-Departure:` trailer in the range and no declared PATTERN change in the directive, so nothing to check there.
