---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/configsync/configsync.go:1044
    summary: An unparseable or non-mapping template base fails every reconcile with an error that names the hub file, not the base, and the walk retries and fails the same way on every start verb.
    fix: In ReconcileHubWideAt's seed-only branch, make a base that does not parse as a mapping fail as a FileError whose Path is configengine.TemplateBaseFile(boardDir, m.Name) and whose message names deleting that file as the way forward, so the next reconcile treats the file as having no base. This needs FollowTemplate's base error told apart from the file's, for example by parsing the base in configsync before the call. Add a row in TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate with a corrupt base asserting the FileError path and an untouched hub file.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configcli/configcli_integration_test.go:370
    summary: Committing the template base by lym config reconcile --layer default --apply, lym fabric reconcile and clone has no test, although the record requires the base committed on Bolt with the file by every reconcile.
    fix: In the --layer default integration test add an --apply run that asserts profiles.template-base is in the Bolt commit beside profiles.yaml, and add one assertion each for lym fabric reconcile (internal/fabriccli/fabric.go:716) and clone (internal/fabriccli/clone.go:66) that the base is committed or recorded as written.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/configcli/configcli.go:629
    summary: The reconcile envelope keys updated, differs and unsettled are asserted by no test; the dry-run test checks only removed and the absence of differs on loom.
    fix: Extend the profiles scenario of TestConfigOverRealHub to plant an operator-set value the template holds otherwise and a former-name pair, then assert both values appear under differs and the pair under unsettled in the dry-run envelope.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubreconcile/walk.go:400
    summary: followedLines and notifyTemplateFollowed are tested only for the removed line; the added, updated and kept-the-hub's lines and the no-mail case are unpinned.
    fix: In TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile, or a table beside it, add cases asserting the orch mail holds an updated line, a kept-the-hub's line with both values, and that a hub whose file already equals the template sends no mail.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/yamlengine/follow_test.go:25
    summary: No FollowTemplate row uses a sequence, although the doc comment and the record say a list is compared whole as one unit.
    fix: Add two rows to TestFollowTemplate, one where a list at the base's value takes the template's new list, and one where a list the operator edited is kept and listed under Differs.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/configsync/configsync.go:244
    summary: An unreadable template base, and the restore of a base written before a later failure of the pass, are untested.
    fix: Add a walk failure case through the afterHubWideWrite hook asserting the written base is restored to its prior bytes or removed, and a configsync row with an unreadable base asserting a FileError naming the base path.
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/yamlengine/follow.go:75
    summary: A changed file is re-encoded with yaml.v3's default width of 80, which folds the long flow mappings of profiles.yaml across lines and changes the operator's layout, against the doc comment's promise to keep each node's style.
    fix: In FollowTemplate set the encoder's width to no wrapping, the way a flow line is kept whole, and add a TestFollowTemplate row whose flow mapping runs past 80 columns and must come back on one line. Confirm blank lines between entries survive or state in the doc comment that they do not.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: internal/configsync/configsync_test.go:908
    summary: TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate was inserted between the doc comment of TestReconcileHubWideAt_MigratesLegacyFabricConfig and its function, so that comment runs into the new test's doc and the migration test has none.
    fix: Move the comment block that begins "TestReconcileHubWideAt_MigratesLegacyFabricConfig pins" back directly above func TestReconcileHubWideAt_MigratesLegacyFabricConfig, and leave the new test's own comment above the new test.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/configcli/hubwide.go:163
    summary: The reconcileDefaultLayer doc and configcli/doc.go:92 still say a present board file is never filled, which a seed-only file now contradicts, and the record asks the docs to drop that wording.
    fix: In the hubwide.go:163 comment say that a non-seed-only base is never filled and that a seed-only board file follows its template against its template base. Scope the sentence "It never fills a base" at internal/configcli/doc.go:92 to hub bases that are not seed-only in the same way.
---

### [MEDIUM] A corrupt template base wedges every reconcile and names the wrong file

**Location:** internal/configsync/configsync.go:1044, internal/yamlengine/follow.go:43
**Issue:** `FollowTemplate` parses the base and the hub file in one call, so a base that is not a YAML mapping returns `parse base YAML`, and `ReconcileHubWideAt` wraps it with the hub file's path at line 1117. The operator is pointed at `profiles.yaml`, and no message names the base or says to delete it. The walk aborts and leaves the build stamp stale, so every start verb retries and fails the same way. The base is a tracked file that two machines or a merge can corrupt, which the record's "a failure to read or write it leaves the hub file untouched" does not cover.

### [MEDIUM] Committing the template base outside the walk is untested

**Location:** internal/configcli/configcli_integration_test.go:370
**Issue:** Record rule 1 asks for the base committed on Bolt with the file, and rule 5 for a clone and `lym config reconcile --layer default` to apply the updates. Only the walk's commit is asserted. The `--apply` path, `lym fabric reconcile` and clone reach the base through `ChangedFilesRel` and a clone `rec.Append`, and a regression there would leave an untracked base with no failing test.

### [LOW] Envelope keys updated, differs and unsettled are unasserted

**Location:** internal/configcli/configcli.go:629
**Issue:** Record rule 5 names `updated` and `differs` in the dry run. The integration test checks `removed`, and that `loom` has no `differs`; a misnamed or dropped `updated`, `differs` or `unsettled` key would not fail it.

### [LOW] Most mail lines of the orch notice are unpinned

**Location:** internal/hubreconcile/walk.go:400
**Issue:** The walk test asserts the `removed` line only. `followedLines` has an `added`, an `updated` and a kept-the-hub's branch, and `notifyTemplateFollowed` sends nothing for no lines; none is reached by a test.

### [LOW] No FollowTemplate row uses a sequence

**Location:** internal/yamlengine/follow_test.go:25
**Issue:** The focus and the doc comment name lists compared whole, and the table has no row with a list for the update or the kept case.

### [LOW] Base read failure and base restore are untested

**Location:** internal/configsync/configsync.go:244
**Issue:** `readTemplateBase`'s read-failure branch and the restore of a base written before a later failure of the pass (`snapshotConfig`, `snapshotBoardConfig`) have no test, although the focus asks whether a failure leaves the hub file untouched.

### [LOW] Re-encoding folds long flow mappings

**Location:** internal/yamlengine/follow.go:75
**Issue:** yaml.v3 v3.0.1 turns the encoder's default width 0 into 80 (`emitterc.go:331`) and breaks flow items past it (`emitterc.go:673`, `emitterc.go:697`). A followed `profiles.yaml` has its roughly 170-column `loom: {...}` lines folded, and the doc comment says each node's style is kept. The tests check only substrings under 80 columns, so none catches it. I did not run an encode, so the exact output is unconfirmed.

### [LOW] A new test split a doc comment from its function

**Location:** internal/configsync/configsync_test.go:908
**Issue:** The comment that opens `TestReconcileHubWideAt_MigratesLegacyFabricConfig pins ...` now sits above `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`, and `TestReconcileHubWideAt_MigratesLegacyFabricConfig` at line 1091 has no doc comment.

### [LOW] Two docs still say a present file is never filled

**Location:** internal/configcli/hubwide.go:163
**Issue:** `reconcileDefaultLayer` says it "never fills a present one", and `internal/configcli/doc.go:92` says "It never fills a base", right after the paragraph that says a seed-only file now gains units. The record asks the reconcile docs to replace the old wording.

## Rejected

- Lens code-tests F6, the configsync rows that repeat `TestFollowTemplate` rows: each row also asserts `Applied`, `WroteTemplateBase`, the dry run's writes and the `Result` fields at the reconcile layer, which `TestFollowTemplate` does not reach, so they are rows of one test and not a duplicate.
- Lens code-tests F4 and F1 merged as above: the unreadable-base and restore gaps are one LOW finding, F6.
- Record rule 3's "template as it stood when the file was seeded where that can be told": the diff takes only the fallback branch, and the loomyard outcome (`code_reviewer_fan: ""` added to `docs` and `speed`, `light` left alone) matches the record either way, so no finding.

## Focus departures

None. Every focus line was reached: the merge's rule 2 cases (`follow.go`, correct by the lens walk and the `TestFollowTemplate` rows), the base's write, commit and read paths (F1, F2, F6), the no-base path and loomyard outcome (correct), seed-only coverage (`profiles` and `models`, accurate in the summary), reporting (F3, F4), the template and `migrate.go` change (correct), docs (F9), and test economy (F5 and the rejected duplicate rows).
