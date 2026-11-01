```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: fix-consistency lens over the F1/F2/F3 change set (`PATTERN.md`, `internal/pauserecord/hold.go`, `internal/pauserecord/doc.go`, `internal/battencli/cleanup_test.go`).

I checked the twins of the changed text:
- **Admit and Network docs against `internal/pauserecord/doc.go`:** both now say the pass-through is process-wide and holds for goroutines and repositories other than the admitted caller. Neither contradicts the `PATTERN-remote-refs-lock` text.
- **NetOp and `doc.go` remote-refs wording:** both describe a lock hold from before the lock is taken until its release, covering the git calls and retries made under it.
- **Other docs and comments that say "operations in flight":** none describes a marker as exactly one call, so none is left stale. These are `tenderengine/doc.go:83`, `tenderengine/report.go:95` and `:124`, `tenderengine/resume.go:46`, `hibernateengine/doc.go:70` and `tendercli/cli.go:296`. `docs/overview.md:476` says the same and also stays accurate.
- **The `remote-refs` op name:** the only `Admit` caller is `gitrepo/remoterefslock.go:30`, which passes `remoteRefsGateOp`, so the new NetOp wording matches how the marker is written.
- **`cleanupStubs.committedErr`:** the field is nil in every other test, so the existing rows are unchanged. The new test uses `removeRunRecords`'s real error path, and the gate was green for both packages.

Tool calls:
1. `grep -rnE "operation in flight|operations in flight|pauserecord\.Admit|every network call of the process" --include=*.go --include=*.md .` (non-test hits): to find twins of the changed wording and the `Admit` callers, which my context lacked.
