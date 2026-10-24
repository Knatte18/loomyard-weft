```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set. Result: no findings.**

Per change, the set I found:

1. **Removed or absorbed PATTERN IDs (about 70 entries): the live citations are covered.**
   - I compared every `PATTERN-<id>` string in the working tree against the 30 IDs in `PATTERN.md`. I left out `.git`, `.lyx`, `_lyx`, `docs/research`, `docs/benchmarks` and `testdata/`, which the record excludes.
   - No code, doc, contract, stencil or crucible file cites a removed ID.
   - The IDs that matched no live entry are not citations of removed entries:
     - Test fixture strings: `PATTERN-new`, `PATTERN-old`, `PATTERN-example-entry`, `PATTERN-sample`, `PATTERN-long`, `PATTERN-one` and similar, in `internal/discussionparser`, `internal/loomengine`, `internal/shedadapters`, `internal/burlerengine`, `internal/pattern` and `internal/loomshed`.
     - Generic placeholders: `PATTERN-change` and `PATTERN-id` in `internal/loomcli/decision.go`, and `PATTERN-entry-id` in `contracts/specs/loom-plan-spec.md:309`.
     - Lens name: `PATTERN-fit` in `contracts/stencils/burler/burler-cluster-fan.md:18` and `internal/burlerengine/doc.go:278`.
   - `docs/overview.md` after the merge carries only live IDs. I checked the retargeted links: `cwd-resolution`, `pair-teardown`, `shed-driver`, `test-isolation`, `parent-directive`, and `cli-cobra`, which survives.
   - Moved rules are cited as package documentation, for example `internal/agentname` and `internal/loomcli`.

2. **`PATTERN-role-skills-typed` citation in `docs/overview.md`: covered.** The diff drops the `[PATTERN-role-skills-typed](../PATTERN.md)` link and keeps `[PATTERN-parent-directive](../PATTERN.md)`.

3. **`recipe-owns-failures` moved to `internal/landingshed/doc.go`: covered.** The section is at lines 7–8 and no live file cites the old ID. The parent's change from "last failing tier" to "first failing tier" at `doc.go:123` and its `docs/overview.md` gate line survived the merge unchanged.

4. **Parent's gate-test tiering change (`internal/gatecli`, `internal/gateslot`, `internal/landingshed/verifygate.go`): covered.** It is not the task's change. The merge kept it, and none of the PATTERN IDs it names were removed.

I made one tool call: a single repo-wide grep and diff of cited `PATTERN-*` IDs against the live set. My context held only the diffs, not the whole-tree citation set.
