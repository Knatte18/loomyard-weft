---
round: 1
focus:
  - "internal/burlerengine/review.go, fix.go, profile.go: both sides edited reviewTable and fixTable (the task added a skills parameter and Profile.LanguageSkills, the parent added Profile.RolePrefix and p.rolePrefix). Settle whether the merged tables, Probe and Resume paths, and every caller pass a consistent role prefix and skill list."
  - "internal/shedrecipe/entries_burler.go and contracts/recipes/loom-recipe.yaml, darn-recipe.yaml: the task added the profile key language-skills and the parent added the role_prefix config key. Settle whether the merged key lists, configRejectUnknown calls, doc comments and the recipe rows agree, and whether every code-round row in both recipes carries both keys where each side's intent requires them."
  - "internal/loomcli/wiring.go: the task loads languageSkills and passes them to burlerengine.New and RunDeps; the parent rewrote loomRerouteCapWayForward and added the Webster plan-conflict route. Settle whether the merged wiring builds every burler and Webster engine with the language skills and whether the wording of the way-forward clause still fits."
  - "internal/websterengine/runlevel.go and doc.go: the task added RunDeps.LanguageSkills and the master Skills append; the parent added RunResult.Gate, RemoveConflictRecord at run start and the conflict record. Settle whether the two edits to Run and RunDeps compose without one assuming the other's absence."
  - "PATTERN.md (changed by the parent) against the task's new language-skills plumbing and the burlerengine.New signature change: settle whether the task's code follows any invariant the parent added or changed."
  - "Test economy in internal/burlerengine/review_test.go, fix_test.go, engine_test.go, internal/shedrecipe/entries_burlerhalves_test.go, internal/loomcli/wiring_test.go and cmd/lyx/testdata/test-budget.yaml: settle whether the merge left two tests covering the same behavior, or left the combined role-prefix plus language-skills behavior uncovered."
---
Seeded from the overlapping files and the two sides' diffs of the burler, shedrecipe, loomcli and websterengine packages.
