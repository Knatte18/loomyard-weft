```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the decision record, tracing the code paths that already run against each decision's state.

**Paths traced, per decision**

- D4, the fenced embedding read by `fencedLines` and its callers (`decisionsInsertOffset`, `locateOpenRisks`, `h2Headings`, `CheckDecisionIDs`, `ParsePatternChanges`, `IdentityText`): **breaks** whenever the body's longest backtick run is under 2.
- D4, `missingSections`, which ignores fences: survives, because the record writes every real heading anyway.
- D4, `decision add`'s darn-only refusals for an uncommitted or changed record (`internal/loomcli/decision.go:383-393`): **breaks**, because the way forward names a path D4 and D6 remove.
- D4, the Drift seat-table subject (`internal/loomcli/wiring.go:785`, `driftwiring.go:129`) against the row `subject` key (`shedrecipe/entries_drift.go:84,144`): **breaks** if only the recipe row changes.
- D4, the Drift records commit through `CommitAnchoredPaths`: survives.
- D5, `validate-discussion`'s recipe-name branch (`internal/loomcli/validate.go:75-77`): survives, since D5 replaces it.
- D6, the stop marker against cleanup of the ephemeral run directory: survives, because Darn-Write removes the marker before each spawn and reads it after the session.
- D7, PR-Review's `runMaterial`, which lists `board-entry.md` (`internal/landingshed/prsession.go:158`): survives, because it stats each path and skips a missing one.
- D7, `NewDarnDiscussionGate`'s snapshot commit: survives, because D5 makes the gate validate-only.
- D10, `CheckSeed`'s exported verdicts and the status spec: survives, since D10 updates `loom-status-spec.md`.
- D12, the plan cap read in `internal/loomengine/caps.go:41`: survives if rewired; that is call-site enumeration and not flagged.
- D12, `moveRunSettings` and `PrepareRunSettings` translating `loom.plan_*`: survives.
- D13, `PrepareRunSettings` running move, then `module.Migrate`, then `checkLocalDocument` against D13's check placed "before ValidateLocal": **at risk**.
- D13, the generic reconcile prune of `RetiredKeys` (as darn's are pruned): survives.
- D13, `childDriverKind` and `ResolveDriverKind`: survives with the override removed (shed.yaml, else llm).

**Findings**

1. BLOCKING, design, D4. The rule "a fence one longer than the longest backtick run" makes a fence that no parser path reads as a fence.
   - `fencedLines` (`internal/discussionparser/append.go:349-361`) opens a fence only on a line starting with three backticks or tildes, and CommonMark also requires at least three.
   - A body with inline code only (longest run 1) gets a two-backtick "fence", and a body with no backticks gets one backtick. This task's own board body is the first case.
   - Every fence-aware reader then sees the body's lines as record lines: `decisionsInsertOffset` and `locateOpenRisks` take a body `## Decisions` or `## Open risks` as the record's own section, and `h2Headings` and `CheckDecisionIDs` count body H2s.
   - Fix: the fence length is at least three, and at least one longer than the longest backtick run.
   - The acceptance row should add a body with no backticks and a body with inline code only.
2. MEDIUM, design, D4 and D6 against `decision add`'s darn-only refusals (`internal/loomcli/decision.go:75`, `383-393`).
   - On a darn run, `decision add` refuses a record that is uncommitted, differs from its committed copy, or dropped a history entry. Each refusal gives `decisionWayDarnWriter`'s way forward: "Darn-Write has not finished its record or is editing it… message the running Darn-Write session".
   - Under D4, Record-Write writes and commits the record. A crash between Record-Write's write and its commit leaves an uncommitted record, and no Darn-Write session exists to message.
   - Under D6, the writer never edits the record, so this way forward points at a route that no longer exists. That breaks PATTERN-refusal-way-forward.
   - The record names only the missing-record refusal (D4) and is silent on these three.
   - Fix: D4 states their new way forward, which is to resume or step Record-Write so it commits, and drops the Darn-Write wording.
3. MEDIUM, design, D13 against the order inside `PrepareRunSettings` (`internal/configreg/runsettings.go:46-86`).
   - The function applies `module.Migrate` before `checkLocalDocument`.
   - Loom's `MigrateConfig` drops every `retiredLoomKeys` entry through `yamlengine.DropKeys` (`internal/loomengine/config.go:709`).
   - D13 places its own check "before" `ValidateLocal`, so a key also listed for Migrate's drop is removed silently from a profile or `run_settings` before the check sees it. That is the silent ignore D13's rationale rejects.
   - Fix: D13 says the check runs before `module.Migrate`, or that the keys never join Migrate's drop list. Reconcile prunes `RetiredKeys` generically, as darn's `verify` and `pr_review` are pruned (`internal/darnengine/config.go:82`, `104-108`).
4. LOW, design, D4: the darn Drift subject is decided by the wiring, not by the recipe row D4 changes.
   - The Drift agent's prompt subject comes from `tables.subject` (`internal/loomcli/driftwiring.go:129`), which `internal/loomcli/wiring.go:785-788` sets to `board-entry` for any darn run by recipe name.
   - The row's `subject` key reaches only `driftengine.Row` (`internal/shedrecipe/entries_drift.go:84,144`).
   - Setting `subject: decision-record` in `darn-recipe.yaml` alone leaves the agent reading the board entry, and D4's acceptance row, "darn's Drift rows read the record as their subject", can pass on the recipe file alone.
   - D8's general retirement of `RecipeDarn` branches covers it only implicitly.
   - Fix: D4 states that the seat table takes the row's `subject`.

**Tool calls**

1. `shedrecipe/entries_drift.go` and the `driftwiring.go` subject grep: to see whether the Drift subject comes from the row config or the wiring, which the manifest lacked.
2. `landingshed/prsession.go` and the `BoardEntryPath` grep: to check whether PR-Review reads the board-entry snapshot D7 stops writing.
3. Grep for `budget.` entry-gate readers and the plan budget key readers: to find what reads the D12 moved keys.
4. `darnengine/config.go` spec, `RetiredKeys` and `MigrateConfig`: to see how darn's retired keys are pruned and checked locally, for D13.
5. `loomcli/decision.go` darn branch: to read `decision add`'s darn-only refusals and their way forward, which D4 and D6 touch.
6. `loomcli/driftwiring.go` `BaselineTable`: to confirm the seat table uses `tables.subject`, not the row's subject.
