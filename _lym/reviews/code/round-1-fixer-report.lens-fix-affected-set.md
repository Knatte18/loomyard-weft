```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set searches for F3 (`ErrTaskPathTaken` text consumers) and F4 (`ErrStrayPathKept` sources and handlers) and check the fix covers every member.

Findings:

- **LOW** — `internal/battencli/wire.go:466-467`, `teardownRefusal`: the message says "the pair for %q was removed", which is false for an `Add` that returns `ErrStrayPathKept`.
  - It is a teardown-only handler, so only a teardown error reaches it. No add path calls it, so the fix introduces no wrong message today.
  - The `ErrStrayPathKept` comment in `remove.go:52` now says "or never made". It matches the sentinel text, which still reads "no worktree stays at the pair's task worktree path".
  - I found no caller that maps an `Add` error through `ErrStrayPathKept`. This is a check note, not a defect.

- **LOW** — `internal/fabricengine/doc.go` still has a stale bullet in the `ErrStrayPathKept` sources list.
  - The review (F4) asked for `Add` to be added "near the Remove and RemoveStrayTaskPath bullets", and the fix did that at `doc.go:793`.
  - `internal/pairteardown/doc.go:95` and `teardown.go:62` describe `ErrStrayPathKept` only for the teardown path. They are not sources of the error, so no change is needed.
  - Coverage of the set (`add.go:179`, `doc.go:793` and `1005`, `remove.go:52`) matches the review's list: every site it named is updated.

- **ErrTaskPathTaken text consumers (F3)**: `grep` for the old "already exists" wording found no remaining reference in `.go` or `.md` files outside `addadoptonly.go`. The `battencli` consumers (`wire.go:393`, `readoptstop.go:31` and `readoptstop.go:140`) use `errors.As` on the type and do not match the message text. The set is covered.

I found nothing else, so the only items are the two LOW notes above and neither needs a change.

Tool calls:
1. `grep` for `ErrStrayPathKept` across `.go` and `.md` files, plus `wire.go` lines 455-480: to list every source and handler of the sentinel. My context did not hold the `pairteardown` or `battencli` files.
2. `grep` for "already exists" in `battencli`, `fabriccli` and `pairteardown`, and for `ErrTaskPathTaken` in `.md` files: to find consumers of the old error text. My context held no search of those packages.
