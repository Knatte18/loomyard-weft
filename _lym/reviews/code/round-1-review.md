---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/store.go#UpsertTasksBatch
    summary: A batch upsert naming recipe arras is never refused, for a new or an existing entry, because validateWrite reads the prior entry from the projected snapshot, which already holds the incoming entry.
    fix: "In internal/boardengine/store.go, have UpsertTasksBatch run checkOfferedRecipe for each projected slug against the entry s.tasks held before the batch (nil when s.tasks has no entry of that slug), not against snapshot. Either call checkOfferedRecipe(incoming, priorFromStoreTasks) in UpsertTasksBatch's validation loop and keep validateWrite's call for the other paths, or pass the pre-write prior into validateWrite explicitly. Add a row to TestUpsertRefusesTheArrasRecipe in internal/boardengine/store_test.go that drives UpsertTasksBatch with a new entry and with an existing darn entry each naming \"recipe\":\"arras\", asserting a refusal naming \"recipe\":\"motif\" and that the stored recipe stays unchanged."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-affected-set
    location: internal/battencli/wire.go:616
    summary: "Error text and config comments still offer arras, though the board entry says nothing visible names it: help text and recipe listings."
    fix: "Drop arras from each listing a user sees, keeping the code that runs it: in internal/battencli/wire.go:616, leave RecipeArras accepted by the check at :610 but remove it from the list of types a Board task may take in the error text; in internal/shedrun/seed.go:109, list the offered recipe names without arras in the unknown-recipe message while still accepting arras; in internal/runprofile/template.yaml lines 2 and 4, drop arras from the recipe list and remove the comment line on the arras block (keep the arras keys themselves, which a running arras run still reads), and mirror the same comment change in internal/configsync/testdata/profiles-flat.yaml if a test compares it with the template; in internal/loomengine/template.yaml:52, describe the split reviewer without naming the arras recipe."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/raddle/raddle-template-agent.md
    summary: "The new Step 3 and Step 4 rules tell the agent to tell the operator, or to write in the text around the table, but in auto and perch mode the rendered mode_rules say no operator is present, so those notes reach no one."
    fix: "In contracts/stencils/raddle/raddle-template-agent.md, after the Step 3 bullets that say \"tell the operator\" and the Step 4 lines that say \"say so to the operator\" and \"In the text around the table\", add one line saying that when no operator is present those notes go in the proposal's own text fields (the part's contract.takes or cost_line, or next_best's line) instead, so a perch-mode split perch reads them; keep the study's wording otherwise."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/role-autonomy.md
    summary: "The split-ready step says When the run waits for that approval but never names the raddle waiting line that tells the orch so, and the orch role's list of what reaches it omits that line."
    fix: "In contracts/orchrole/role-autonomy.md's section A Sketch a motif run left, say that the run waits for the approval when the orch receives the line opening `raddle: the proposal for the Sketch <slug> passes its checks and waits for the operator's approval`; in contracts/orchrole/orch-role.md's Notices list, add that line as one that reaches the orch, to put to the operator."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/dobbyengine/template.yaml:6
    summary: "The land_notice comment says the setting alone decides whether dobby posts a line once a run lands, and the autonomy comment says the orch reaches the operator only for a severe halt or a merge-ready beam; both are now false."
    fix: "In internal/dobbyengine/template.yaml, extend the land_notice comment to say a landed motif run's line, saying its entry is now a Sketch, goes whatever the setting reads, and extend the autonomy comment's list to a severe halt, a merge-ready beam or a raddle split's approval."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:1706
    summary: The overview lists arras among the recipes a Board task's recipe takes, and the glossary calls arras the recipe a board task names for a tapestry-sized note, which the board now refuses.
    fix: "In docs/overview.md around lines 1706-1710, say arras is no longer offered on the board and that a tapestry-sized note runs motif; in docs/reference/weaving-glossary.md:133, change the arras row's note to say the recipe is kept but no board entry may be given it, and that motif serves a tapestry-sized note."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbypairanchor_test.go#TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitTheEndedEntryThePairAndTheMotifRecipe
    summary: The new child-recipe read in DobbyRuns has an error path, which fails the whole listing, and no test row reaches it.
    fix: "In internal/beamcli/dobbypairanchor_test.go, add a case that plants a directory at shedrun.ChildRecipeFile(prime, <slug>) so ReadChildRecipe fails for a reason other than a missing file, and assert what DobbyRuns returns for it (an error naming the run), as the unreadable landed-commit case does at :295."
---

### [BLOCKING] A batch upsert gives an entry the arras recipe unrefused

**Location:** `internal/boardengine/store.go#UpsertTasksBatch`

**Issue:** `UpsertTasksBatch` builds `snapshot` with every patched or new entry in place (`store.go:1109-1119`) before it calls `validateWrite(snapshot, incoming)` (`store.go:1132`).
`validateWrite` then hands `checkOfferedRecipe` the snapshot's own entry of the slug as `prior`, which is the incoming entry, so `prior.Recipe == arras` holds and the check returns nil.
Every batch write naming `"recipe":"arras"`, new entry or existing, takes the recipe, so the board entry's "A board write naming `recipe: arras` is refused" fails on `lym board` batch writes.
`TestUpsertRefusesTheArrasRecipe` drives `UpsertTask` only, so it does not catch this.

### [MEDIUM] Error text and config comments still list arras

**Location:** `internal/battencli/wire.go:616`

**Issue:** The board entry says nothing visible names arras, including help text and recipe listings.
The unsupported-child-recipe error at `wire.go:616` tells the user a Board task's type may be `arras`.
`shedrun/seed.go:109` lists `RecipeNames()`, arras included, in its unknown-recipe message.
The seeded `profiles.yaml` comments (`runprofile/template.yaml:2,4`) and the `loom.yaml` split reviewer comment (`loomengine/template.yaml:52`) describe an arras recipe a user may configure, which the `lym config` editors show.

### [LOW] Raddle notes addressed to an operator in modes with none

**Location:** `contracts/stencils/raddle/raddle-template-agent.md`

**Issue:** The new rules say "tell the operator which and why", "tell the operator what to file by hand", "say so to the operator" and "In the text around the table, say which project … the filed entries land under".
The same prompt's `{{.mode_rules}}` (`internal/loomengine/raddletable.go:63-74`) say no operator is present in auto and perch mode, and only interactive mode shows a table with text around it.
In those modes the agent meets a contradiction and the notes reach no one, so the perch cannot check what the rubric's new bullets judge.
The wording is the study's, so the fix adds where the notes go rather than changing it.

### [LOW] The split-ready step never names the line that triggers it

**Location:** `contracts/orchrole/role-autonomy.md`

**Issue:** "When the run waits for that approval, notify `[operator] split ready <slug>:`" names no signal.
The signal is `raddle-propose`'s waiting notice to the parent orch (`internal/loomcli/raddleverbs.go:129`), and the orch role's Notices list does not name it either, so an orch under autonomy cannot tell which line to act on.

### [LOW] Dobby's config comments contradict the new notice and autonomy rules

**Location:** `internal/dobbyengine/template.yaml:6`

**Issue:** The `land_notice` comment says the setting decides whether dobby posts a line once a run lands, but a landed motif run now gets its Sketch line whatever the setting reads.
The `autonomy` comment on the next line says the orch reaches the operator only for a severe halt or a merge-ready beam, while `role-autonomy.md` now adds a raddle split's approval.

### [LOW] Overview and glossary still offer arras for a board task

**Location:** `docs/overview.md:1706`

**Issue:** `docs/overview.md:1706-1710` lists `arras` among the recipes a Board task's `recipe` takes.
`docs/reference/weaving-glossary.md:133` calls it "the recipe a board task names for a tapestry-sized note".
The board now refuses both of those uses.
The change description says these entries stay, but keeping the recipe's code does not require the docs to offer it.

### [LOW] The child-recipe read's failure path has no test

**Location:** `internal/beamcli/dobbypairanchor_test.go#TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitTheEndedEntryThePairAndTheMotifRecipe`

**Issue:** `DobbyRuns` returns the error of `shedrun.ReadChildRecipe` (`internal/battencli/dobbyreads.go:230-234`), which ends the call for every run that tick.
That handling matches the sibling reads, but no row plants an unreadable child-recipe record, so the new error path has no coverage.

## Rejected

- code-scope's finding 2 (the orch promotes the Sketch at once with `depends_on` rather than "once nothing it waits for is open"): the entry also asks for `depends_on` "for what it waits for, so the board shows why", which only means something when the Sketch is promoted while it still waits. A promoted Sketch is released, and dobby starts it only once those entries close (`boardcli/cli.go:192-196`). The diff's reading builds the entry's intent, so this is no deviation.
- code-correctness's finding 2 (an entry demoted to a note keeps the arras exemption): this is the rule as stated, and the lens itself reports it as no defect.
- The `outcome` read when the child-recipe read fails, which ends the whole listing: it matches the sibling reads in the same read set and is no new failure mode. Only its missing test is kept, as F7.

The `Merge danger` call (two-way, blast radius orch) holds against the diff.
The refusal changes no stored format, and an older binary reading the board meets entries it already reads.
The Sketch line is a new creel text, and no other binary parses it.
