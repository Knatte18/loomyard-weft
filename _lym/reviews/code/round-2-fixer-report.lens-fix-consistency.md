```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for consistency with its surroundings (fix-consistency lens).

Findings:

1. **LOW** — `_lym/landing/summary.md`, "Design decisions", first bullet (changed text: "The status commits stage the settle records; the other files … stay uncommitted until a later records commit or the run's removal").
   - The same bullet's opening sentence still lists "settle records" among the files the root holds and exempts.
   - Settle records are now committed by the status commits, so listing them in the exempt set is slightly in tension with the new sentence.
   - The tension is mild, because the exemption still covers the window between the settle-log write and the next status commit.
   - The `preflight.go` and `doc.go` comments share the same shape.
   - The summary's intro (line 7) and "Before" paragraph correctly describe the pre-fix state. I found no other twin left stale.

2. **LOW** — `internal/preflight/doc.go:25-29` and `preflight.go:17-21` (changed text: "until a later records commit or the run's removal").
   - I did not verify what the "later records commit" is, or that anything commits the stop markers at all.
   - The review said nothing commits them in a task pair; `parentRecordsCommitter` commits the parent's root only.
   - The wording is hedged, so it states no false mechanism.
   - "A later records commit" is vague enough that a reader may assume a task pair's commit exists.

Checked and consistent:
- `shedrun.SettleRecordsRel` exists at `paths.go:246`.
- Both status commits stage it: `wiring.go:117` and `battencli/commitstatus.go:77`.
- Both have test rows: `wiring_commitstatus_test.go:300` and `commitstatus_test.go:176`.
- The `paths.go:344` `SettleRecords` comment now matches this.
- The `statusCommitPathspec` doc at `wiring.go:85` already lists the settle records, so it needs no edit.
- No `pairteardown` claim remains in `internal/preflight` or `internal/shedrun`; the grep showed only unrelated `fabricengine` mentions.
- The `RunsRootRel` comment no longer enumerates consumers.

Tool calls:
1. Bash grep for `SettleRecordsRel` in `internal`, and for pair-teardown mentions in `preflight`, `shedrun` and `fabricengine`. My context lacked the status-commit code, the tests and any leftover teardown claims.
