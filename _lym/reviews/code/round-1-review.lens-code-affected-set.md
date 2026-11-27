```yaml
lens: code-affected-set
outcome: reported
```

Task: work the code-affected-set lens over the diff.

**Affected set, per change**

1. **`ForkReport.SpawnedBySubagent` added** (`internal/shuttleengine/forkaudit.go:56-57`)
   - Readers: `burlerengine.auditFixRound` (`cluster.go:942`) reads it and is covered.
   - `websterengine.CheckFork`, `ForkWarnings`, `NewTranscripts`, `ClassifyAttribution`, `recordbatch.go:379-410` and `:792-815`, and `shuttleengine/wait.go:2082` don't read it.
     None of them needs to: webster bans every Agent call by a fork, so whichever fork spawned the subagent is flagged anyway.
   - `auditClusterRound` and `forkViolation` don't read it. Their behaviour is unchanged by the diff.
   - The `shuttlefake` fake and test literals still compile, since a field was added and none removed.
   - Type doc: the `ForkAudit` doc line `forkaudit.go:26`, "Forks holds one report per subagent the parent spawned", is now inaccurate. The new field says some reports are for subagents another subagent spawned. Not covered (finding 3).

2. **`auditFixRound` exempts `SpawnedBySubagent`** (`cluster.go:942`)
   - Callers: `fix.go:347` gets the right count.
   - Doc: `burlerengine/doc.go:111` gained a line, so that is covered.
   - `fixViolationWayForward` (`cluster.go:902-908`) works out "no target change was found: no fork wrote a file…" from `forkViolation`'s `untouched` flag, which is computed over `selfCheck.Forks` only (`cluster.go:864-871`).
     A fork that a self-check fork spawned is now dropped from that set. If it writes, the halt names the self-check fork's nested spawn but still says no target change was found. Not covered (finding 1).
   - Stencil: `contracts/stencils/burler/burler-fix-self-check.md:48` says "A fork spawns nothing, so the fixer is the only session that spawns." That contradicts the decision that the fixer's fresh subagent may spawn forks. Not covered (finding 2).

3. **`auditForkTranscript` signature change and inherited-id exclusion** (`audit.go:367`)
   - Callers: `AuditForksIncremental` and the tests `audit_spawnboundary_test.go`, `audit_streaming_test.go` are all updated, so covered.
   - Fixture: `TestAuditForks_WriteEventsCarryTimeAndResult` copied the parent fixture as the fork transcript. It now rewrites the ids (`audit_test.go:205-218`), so covered.
   - Other tests that copy a parent fixture as a fork transcript are not in my context. The gate decides whether they still pass.

4. **`auditParentTranscript` now returns a struct**
   - The only caller is `AuditForksIncremental`. `audit_parentfacts_test.go` may call it directly, but the gate covers compilation.

5. **New `.meta.json` key `parentAgentId`**
   - Only the audit reads it.
   - Fixtures: only `audit_nested_test.go` carries it, and the `spawnboundary` and `audit_test` metas don't need it. Covered.

6. **Rules**
   - The constraints declare no PATTERN changes, so there is nothing to land.
   - Commit messages: the `Root-Cause:` trailer is present on `fa94897ed` and names the defect the diff removes. No `Pattern-Departure:` or `Loosened-Check:` trailer is needed.
   - `docs/benchmarks/test-redundancy.md` lists audit tests, but it is a measurement report and exempt.

**Findings**

1. **MEDIUM, design.** `internal/burlerengine/cluster.go:938-947`: the fix-round way forward can falsely say "no target change was found".
   - Cause: `untouched` is now computed without the forks that self-check forks spawned. A file written by a fork a self-check fork spawned is left out of `forkViolation`'s target-change check, so the halt tells the operator to resume when it should say revert.
   - Evidence: `cluster.go:864-871` (`untouched` over `audit.Forks`), `cluster.go:906`, and `cluster.go:940-943` (selfCheck filter).

2. **LOW, consistency.** `contracts/stencils/burler/burler-fix-self-check.md:48`: "A fork spawns nothing, so the fixer is the only session that spawns" contradicts the diff's decision that a fixer's fresh subagent may spawn forks (`doc.go:111`, summary "Design decisions").
   - Evidence: `burler-fix-self-check.md:48`.

3. **LOW, consistency.** `internal/shuttleengine/forkaudit.go:26`: the `ForkAudit` doc still says Forks holds "one report per subagent the parent spawned". Since the diff added `SpawnedBySubagent`, Forks visibly also holds subagents that other subagents spawned.
   - Evidence: `forkaudit.go:26` and `forkaudit.go:56-57`.

**Tool calls**
- grep across docs, stencils and Go for the wording of the fork-audit and spawn rules: my context had no stencils or docs.
- grep of the twill and burler stencils and the docs/specs/pattern files for subagent and fork rules and audit names: I needed the fixer's stencil wording.
- sed of `burler-fix-self-check.md:36-50` and grep of the fix and fork-spawn stencils: I needed the full text of the spawn-rule sentence.
