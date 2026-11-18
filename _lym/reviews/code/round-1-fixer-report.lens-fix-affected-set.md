```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for F1, F3, F7 and F8 and check the fix covers every member.

**F1, literal `"fork"` sites: covered.** The only non-test `"fork"` literal left in `claudeengine` is the constant declaration, `background.go:35`.

**F3, `auditParentTranscript` callers and `ForkAudit` consumers: covered.** The function has one caller, `AuditForksIncremental`, and both of its `ForkAudit` returns set `NamedForkSpawns`. No other non-test code builds a `ForkAudit` in the audit path. `websterengine` reads only `NamedSpawns`, as the review says it should.

**LOW — F8 left a stale "fork transcript" wording in `audit.go`.**
- Changed text: the `audit.go` file header and the `AuditForks` and `AuditForksIncremental` docs.
- Wording left: the `AuditForksIncremental` doc says "A missing parent transcript or unreadable fork transcript is an error" (`audit.go:38`). The error string says `read fork transcript %q` (`audit.go:384`). Both sit in the function that now reads every subagent transcript.
- Why it matters: the doc line is in the exact comment F8 reworded.
- Fix: change both to "subagent transcript".

**LOW — F7 did not cover `wait.go:1925`, which still says "audits fork subagents".**
- Changed text: the `ForkAuditor` docs in `engine.go`.
- Wording left: the comment "For fork mode, it audits fork subagents and attaches the result." The same audit now covers every subagent, forks and fresh ones.
- Fix: reword it to "subagents".

I rate the `activity.go`, `usage.go`, `transcriptread.go` and `claudeengine/doc.go:130-136` hits as out of scope. They describe the usage and activity readers, not the audit, and the fix did not touch them. The same goes for `sessionusage.go:15`, `spec.go:50` and the `burlerengine/doc.go` hits.

Tool calls:
1. Grep of `"fork"` literals, `auditParentTranscript(` callers and `ForkAudit{` constructions, run as one Bash call, because my context lacked the repo-wide hits.
2. Grep of "fork transcript", "fork subagents" and "record of fork" in `shuttleengine` and `burlerengine`, because the F7 and F8 wording sets are repo-wide.
3. Read of `claudeengine/doc.go:124-139`, to see whether the package doc describes the audit's fork-only reading. It describes the usage reader, so it is out of scope.
