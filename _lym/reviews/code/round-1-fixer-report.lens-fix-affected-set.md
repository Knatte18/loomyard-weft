```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, rerunning the searches behind the set-premised review findings.

**Left unchecked:** none. I searched for F5 (readers of the fork audit), F4 (sentences saying who spawns), F6 (docs saying what `Forks` holds) and F3 (callers that treat `fs.ErrNotExist` specially).

- **LOW**: the `**Blast radius:**` line in `_lym/landing/summary.md` (F5) still leaves out two readers of the fork audit.
  - Burler's cluster-review audit: `auditClusterRound` runs `forkViolation` over every report of the reviewer's `ForkAudit` (`internal/burlerengine/cluster.go:833-846`, `internal/burlerengine/review.go:47-48,303`). The new line names only "burler's fix-round audit".
  - Webster's run-exit audit cross-check: it runs `CheckFork` over `result.ForkAudit.Forks` and compares their count with the begun fork batches (`internal/websterengine/runlevel.go:1091-1118`). The line names only webster's record-batch audit and run-write history, though "shuttle's fork-authorized run finish" covers where that audit is produced, not this check of it.

No findings for F3, F4 or F6. `contracts/stencils/twill/twill-directive.md:46` has a similar "the writer is the only session that spawns" sentence, but it governs the writer's lens forks, not the fixer this change is about.

Tool calls:
1. `grep` for `AuditForks` and `.Forks` readers: my context held only the readers named in the review and `runwrites.go`.
2. `grep` for sentences saying who spawns: my context had only a narrower pattern's hits.
3. `grep` for docs describing what `Forks` holds: not in my context.
4. `sed`/`grep` on `websterengine/runlevel.go`, `burlercli/run.go` and `burlerengine/review.go`: call 1 showed bare lines without the code around them.
