```yaml
lens: fix-consistency
outcome: reported
```

No findings that reach BLOCKING or MEDIUM. My task was to check the change set's agreement with its surroundings (the fix-consistency lens). I found two LOW stale twins.

- **LOW** — `internal/shuttleengine/claudeengine/audit.go:38`, the `AuditForksIncremental` doc.
  - F8 reworded the "every fork transcript" phrase on the line above.
  - The next sentence still reads "A missing parent transcript or unreadable fork transcript is an error", though the function now reads every subagent transcript.
  - Its twin, the `auditForkTranscript` read-error message, is also left as "claudeengine: read fork transcript %q".
  - `ForkReport.TranscriptPath`'s comment in `internal/shuttleengine/forkaudit.go` still says "fork's transcript".
  - Evidence: F8 asked for subagent wording in the file header and the two function docs, and the same file keeps these three fork-only phrasings.

- **LOW** — `internal/shuttleengine/engine.go:220`, the `AuditForksIncremental` doc.
  - Its list of always-cumulative parent facts, "(SpawnCalls, NamedSpawns, ParentWriteCalls, ParentWrites, ParentBashCommands)", leaves out the new `NamedForkSpawns`.
  - It already left out `ParentWriteEvents`, so this is a pre-existing enumerated-list drift that the new field widens.
  - Evidence: `NamedForkSpawns` is filled from the full parent read in both return paths of `AuditForksIncremental`, so it is always cumulative like its siblings.

Checked and consistent:
- **Return paths:** both `ForkAudit` return paths in `audit.go` set `NamedForkSpawns`.
- **Provider and fake implementations:**
  - Only the Claude engine fills real audits.
  - `shuttlefake` and `fakes_test.go` return canned `ForkAudit` values, so the zero `NamedForkSpawns` default is harmless there.
  - `auditFixRound` has one production caller, `internal/burlerengine/fix.go:336`, which passes the engine's audit through.
- **Other consumers of `NamedSpawns`:** webster and `auditClusterRound` still read `NamedSpawns` unchanged, as F3 asked.
- **Test text:** the new `cluster_test.go` error substring matches the `forkViolation` message as wrapped by `auditFixRound`.
- **Constant doc:** the `forkSubagentType` comment now covers the `.meta.json` agentType and `audit.go` has no stray `"fork"` literal.
- **Moved test:** `forkMeta` stays defined in `audit_spawnboundary_test.go` and `audit_test.go` uses it from the same package, so nothing dangles.
- **Test headers:** both test file headers are consistent after the test move.

Tool calls: 1. A `grep` for `ForkAuditor` implementers and `auditFixRound` callers, because my context did not show whether any other provider or caller needed `NamedForkSpawns`.
