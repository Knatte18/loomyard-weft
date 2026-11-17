```yaml
lens: fix-code-correctness
outcome: reported
```

Fork report (lens: fix-code-correctness). Task: run each changed branch on a concrete input and check callers and the added tests.

**Findings**

1. **LOW** — `internal/tenderengine/bolthold.go`, the new replace branch in `SyncBoltHold` (Lift, then Pause).
   - **Summary:** the replacement is two separate record updates, not one. `Lift` calls `liftEntries`, which gives the lifted entry's remaining sessions a resume line (`resume.go:203-205`) and moves them to a covering entry if there is one.
   - **Evidence:** take p1 with a bolt-hold entry for oslo that holds a session, and a hold that moves to bergen. `Lift` resumes that session, and only afterwards does `Pause` write the bergen entry. The session is told to resume while the hold still stands, and it is held again only once the new entry exists.
   - **Impact:** this is a transient window rather than a wrong end state. The existing drop branch already lifts the same way. Say so in the report if it is accepted.

**Checked, no defect**

- **F3 test row:** it fails with the defect in place. If the skip returned, `p1` would stay recorded for oslo and `Written` would be empty. The row would then fail both the `Written` check and the machine check, which now compares against `tt.req.Machine`.
- **F3 error paths:** a failed `Lift` joins the error and continues, so the old entry stays. A failed `Pause` is handled as before.
- **F3 callers:** the other rows record oslo in both the entry and the request, so the changed machine check still holds.
- **F2:** with `Wrote` false and `Standing` nil, no notice is posted and `Wrote` is returned as before. The new row would fail with the defect in place, because a notice would be posted. The re-run test still holds, since its `WriteHold` fake writes.
- **F1 test:** with plain `a == b`, "osl46" against "neve" returns `Standing` non-nil, so the row fails. Seeding the hold through `WriteHold` before the rename avoids the stale second clone.
- **F5 row:** the same machine with another kind is appended, so two holds stand in order, matching the doc line added to `WriteHold`.

**Out of scope**

A cut between a successful `WriteHold` and the `Notice` call now never posts the notice on the re-run. This follows from the reviewer's proposed fix and is not worth a finding.

**Tool calls**

1. `grep` of `Lift` in `internal/tenderengine`: my context lacked `Lift`/`liftEntries` behaviour, needed to judge the replace branch.
