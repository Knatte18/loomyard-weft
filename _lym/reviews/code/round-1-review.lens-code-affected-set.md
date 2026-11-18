```yaml
lens: code-affected-set
outcome: reported
```

**Per change, the set I found:**

1. **`ForkReport.FreshSubagent` added; `ForkAudit.Forks` now means every subagent**
   - `burlerengine.auditFixRound` (`cluster.go:548-568`): reads the field. Covered.
   - `burlerengine.auditClusterRound` (`cluster.go:488-501`, called from `handoff.go:44`): ignores it, so the review round still counts every subagent as before. Covered (summary and commit message say "unchanged").
   - `burlercli.resultEnvelope` forkCount (`run.go:249-250`): reads the review's audit only; unchanged behavior. Covered.
   - webster: `NewTranscripts` (`audit.go:456`), `recordbatch.go:370`, `runlevel.go:1085` (CheckFork), `runlevel.go:1109` (audited cross-check), `runwrites.go:54,68`: ignore the field, so behavior is identical to before the diff. Covered.
   - `shedadapters/burlerfix.go:345-346` (`fix_forks` usage): receives the filtered count; its docs (`roundusage.go:45`, `shedadapters/doc.go:424,653`) say "fork count", which is still accurate. Covered.
   - `internal/testkit/shuttlefake`: zero value means fork, which matches the old behavior. Covered.
   - Codex engine: no `AuditForks` implementation, nothing to fill. Covered.
   - `shuttleengine/engine.go:210-218` `ForkAuditor` docs: still say the audit reads "the provider's on-disk record of fork subagents" (`AuditForks`/`AuditForksIncremental`), while they now return fresh subagents too, flagged by the new field. **Not covered** — stale.
   - `claudeengine/audit.go:1-4` file header ("fork subagents", "fork transcripts") and `AuditForks` doc `audit.go:28-29` ("reporting every fork transcript"): the diff added one line to the header but left these; they are now stale against `forkaudit.go`'s widened text. **Not covered.**

2. **`forkSpawnToolUseID` → `readSubagentMeta`/`subagentMeta`**
   - Only caller `auditForkTranscript` (same file). Covered.

3. **`auditFixRound` contract (neither checks nor counts fresh subagents)**
   - `burlerengine/doc.go:214-218`: updated. Covered.
   - `cluster.go:3-4` file header ("applies the same per-fork checks to a twill fixer's self-check forks"): still accurate. Covered.
   - `burlercli/doc.go:11`: still accurate. Covered.
   - Stencil `burler-fix-self-check.md:45,48`: fork boilerplate is read-only and "the fixer is the only session that spawns" — consistent; the diff changes no stencil. Covered.

4. **Repository rules / PATTERN / commits**
   - No declared PATTERN changes in the directive; nothing to land.
   - `PATTERN-shuttle-provider-seam`: meta-file knowledge stays in `claudeengine`; `shuttleengine` gains only a provider-invariant bool. Holds.
   - `PATTERN-tolerant-records`: `subagentMeta` is read with `json.Unmarshal`, never refusing unknown keys. Holds.
   - Commit messages: no `Pattern-Departure:` trailers; none needed. Claim "The review-round audit and webster's audits are unchanged" holds per the consumers above.

**Findings:**

- **LOW · consistency** · `internal/shuttleengine/engine.go:210-218` — The `ForkAuditor` interface docs still describe the audit as reading "fork subagents", but `ForkAudit.Forks` now holds every subagent, with `FreshSubagent` telling them apart. Evidence: `engine.go:213` ("AuditForks reads the provider's on-disk record of fork subagents") vs `forkaudit.go:25` in the diff ("one report per subagent the parent spawned, forks and fresh subagents alike").
- **LOW · consistency** · `internal/shuttleengine/claudeengine/audit.go:1-4,28-29` — The file header and `AuditForks` doc still say "fork subagents" / "every fork transcript", while `auditForkTranscript` now reports and classifies every subagent transcript. Evidence: `audit.go:3-4`, `audit.go:28` vs `audit.go:313` ("reads one subagent's transcript into a ForkReport").

Tool calls:
- One grep/sed over `roundusage`/`shedadapters`/`burlercli` docs, `shuttleengine/engine.go:210-220` and `codexengine` — my context lacked the `fix_forks` doc readers, the `ForkAuditor` interface doc text, and whether Codex implements `AuditForks`.
