```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set (F1–F13) for stale siblings and twins left unchanged, through the consistency lens.

**LOW** — `internal/pairteardown/doc.go`, near line 296. This is a sibling the fix did not update.
- The step-3 sentence was amended to say the owned-process end always spares the orch's provider process.
- A later sentence, "Then it ends the slug's registered processes with the dobby role spared:", still names only the dobby as spared. I saw it earlier in `grep` output at line 296 and did not re-read it.
- This is a LOW, not a contradiction: the later sentence is about the session-end path, and the always-spared set now also holds teardown children (already the case) and the orch.
- Evidence: `endOwnedProcesses` now appends `runproc.RoleTeardown, runproc.RoleOrch` for every caller, including the `skip.go:94` call that passes `RoleDobby`.

Checked and consistent, no finding:
- `EndSubject` has no remaining caller on the old 4-argument signature; the only call is `pairteardown/endsubject.go:40`.
- The "dobby last" wording matches across `runproc/end.go`, `runproc/doc.go` and `PATTERN-child-processes.md`. It is conditional on `endDobby` in all three.
- The orchengine and orchcli docs agree with `ownerTick` on the strand being stopped even when the end fails, and on no retry once the strand is gone.
- `owner.go`'s `OwnerWatch.End` comment, the `orchCLI.endSubject` field comment and `orchcli/doc.go` all say the watcher leaves the dobby running.
- `fabriccli/anchor.go:468` already spares `RoleOrch`, so it needs no twin edit.
- `beamcli/doc.go` Bound says only that `runproc.End` ends the dobby. It does not name the orch, so leaving it unchanged is right.
- The `summary.md` bullets the review names sit in the weft worktree under `_lym`. That is outside this change set, and I did not check them.

Tool calls:
1. Ran `grep` across `.go` and `.md` files for `dobby last`, `retries both`, `endSubjectProcesses` and `EndSubject(`. My context held no repo-wide sibling search.
2. Ran `grep` for `RoleOrch` in non-test Go files, to see whether any other spare or end list needs the orch added. My context held no repo-wide search for that role.
