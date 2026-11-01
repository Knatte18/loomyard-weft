```yaml
lens: fix-consistency
outcome: reported
```

No findings on the whole change set except one stale cross-reference.

**MEDIUM — `internal/fabricengine/doc.go:1214`.** The package doc still says "`resolveArchiveTip` resolves `FETCH_HEAD^{commit}`, revision syntax over git's fetch output." The F5 fix removed that `rev-parse` (`internal/fabricengine/archive.go:160-165`). The function now fetches the SHA that `ls-remote` listed and returns it, so the doc line describes behaviour the code no longer has. It is a bullet in a list of revision-syntax uses, so it needs removing or rewording. The `doc.go:69` and `originreconcile.go:497` mentions of FETCH_HEAD concern the judge-only fetch and still hold.

Everything else I checked agrees:
- **`PATTERN.md` entry:** its exemptions (tag push, `--refmap=` fetch, probe-clone fetch) match the guard's allowlist for `recordsprobe.go` and `originreconcile.go`.
- **Other FETCH_HEAD readers:** `FETCH_HEAD` is read after an unlocked fetch in `recordsprobe.go:183` (a private probe clone) and `remoteleftover.go` (the `--refmap=` probes). Neither is the shared-worktree race F5 describes, so neither is a left-over twin.
- **`internal/gitrepo/doc.go:57`:** it says every fetch, pull and push method runs under the lock, which matches the new `PushRebaseFree` doc.

Calls made:
1. `grep -n FETCH_HEAD internal/fabricengine/*.go` (non-test): to find twins of the F5 pattern and stale mentions of it, which my context lacked outside `archive.go`.
