```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix against each finding's set premise (affected-set lens).

**F1 (set: sites that remove run records and can fail half-way)** — LOW
- Changed text: `restoreMotifRunRecords` (`internal/battencli/cleanup.go`).
- Summary: the other `removeRunRecords` caller that can leave a half-removed run has no restore.
- Evidence: `internal/battencli/cancelcmd.go:294` calls `removeRunRecords` on the same deletion-before-commit path.
- That is `lym batten cancel`, outside the reviewed `lym beam create` path, so this is a note and not a gap in the fix.
- Inside the reviewed path every non-scratch error now restores, via the `case err != nil` branch in `clearMotifRun`.

**F4 (set: places that describe create and cleanup)** — LOW
- Changed text: `internal/beamcli/cli.go:481-500`, the `lym beam create` help `Long` text.
- Summary: the CLI help lists create's steps and says nothing of the motif-run clearing step.
- Evidence: the steps list runs 1, 2, 3 (claim root, record owner, start the pawl run), with no step for removing a done motif run first.
- The fix updated `docs/overview.md:458` and `docs/overview.md:694`. `internal/beamcli/doc.go:26-27` already covers it.
- `battencli/cli.go:12` also already mentions `ClearMotifRun`.
- The `lym batten cleanup` help `Long` text (`cleanup.go`, `cleanupCmd`) does not mention that create also clears a done motif run; the overview paragraph at `docs/overview.md:694` was updated for this.

**F3 and F2** — no findings.
- F3's rows (commit failure, scratch-empty failure, push-spawn failure, untracked run) are all added, plus the retry test.
- F2's assertion sits in the Seed-Child integration step, plus the refused-seed case.

Tool calls:
1. Grep for `removeRunRecords(` callers, `batten cleanup` sites and `motif_run_cleared`/`ClearMotifRun` sites. My context lacked a repo-wide caller and doc search.
2. Grep of `pattern/PATTERN-tolerant-records.md`, `internal/cleanupcli/doc.go` and the `beamcli` refusal wording. My context lacked these doc sites.
3. Grep of the `create` command help in `internal/beamcli/cli.go`. My context lacked the CLI help text.
