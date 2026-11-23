```yaml
lens: fix-affected-set
outcome: reported
```

**Findings**

- **LOW** — `internal/boardengine/board.go:293`, comment of `RemoveCommand`. It reads "the verb takes a JSON payload or a bare handle, never a bare slug."
  - F4 covers the sites that say every lookup verb takes a bare slug. This comment is a member of that set the fix left alone, and it is now false.
  - `payloadLegend` (`internal/boardcli/payload.go:12`) and `doc.go:22` both say a payload naming one entry may be a bare slug.
  - `internal/boardcli/cli.go:1194` also describes only the bare handle. It is a sibling the review did not name, and I did not check what it says.

**Unchecked (cap not reached; these did not need a search)**
- F2, F3: single-function and single-table changes. F2's one caller is `Call` in `recordwrite.go` and the `--body-file` path.

**Checked, no gap**
- F1 (cancel way-forward sites): `recordwrite.go:56` now names `--remove`. The other sites naming `lym batten cancel <slug>` are in other packages and belong to cases other than the envelope reason.
  - `internal/loomshed/samplersession.go:30` and `internal/loomcli/samplerdone.go:79` offer a cancel as an alternative to resuming. They do not describe the envelope.
  - `contracts/orchrole/orch-role.md:67` says an abandoned run is ended with a plain cancel, and line 69 adds `--remove` for a kept pair. That is consistent with the fix.
  - `internal/boardengine/doc.go:186` also names a plain cancel for an abandoned run. That is not the envelope reason, so I count it as out of scope.
- F5 (`--body-file` refusal docs): `internal/boardcli/cli.go` carries the upsert flag line and merge's help, and `internal/boardcli/doc.go` already states the refusal.
  - `docs/getting-started.md:62` mentions the flag without listing refusals.
  - `docs/overview.md:625-627` and `internal/selfreportcli/cli.go` concern `selfreport`, a different verb.
  - The `SKILL.md` flag text I grepped lists no refusals, so nothing there needs the new one.

**Calls**
1. A combined `grep` for "bare handle", `body-file` and `batten cancel <slug>` across the repo, to enumerate each finding's sites. My context held only the files the review named.
2. `sed` on `internal/boardengine/board.go:288-296`, plus a `grep` on the docs for `body-file`, to read the comment and the docs hits in full. The first search gave only match lines.
