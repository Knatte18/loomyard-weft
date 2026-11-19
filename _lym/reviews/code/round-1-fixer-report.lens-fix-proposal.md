```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1–F5 that the fixer carried out as proposed.

**Findings**

1. **LOW** — `doc.go`, layout paragraph: "a rewrite of a marker that stays leaves them."
   - The claim is wider than the code. `commitAnchorMarker` (`anchormarker.go:83-86`) calls `forgetLayouts()` on every commit it makes. That includes a rewrite of an existing marker with different bytes, not only a marker that appears.
   - The case: a worktree whose marker is rewritten from `.` to `sub` through `commitAnchorMarker`. Every kept answer is dropped, which contradicts the new sentence.
   - The reviewer's narrowing in F3 rests on the claim that the other writers (anchor move, `Add`'s staged marker, re-root, `commitEmptySnapshot`) do not call `forgetLayouts`. It never checked the rewrite case that `commitAnchorMarker` itself covers.
   - A safer wording is "a fabric write that adds or retires a marker drops every kept answer". A rewrite may or may not drop them, and either way the answer stays correct.

2. **LOW** — `mergestateactive.go`, line 13: "a cherry-pick, revert or rebase there does not count, since fabric's own weft replay cherry-picks under the write lock the commit waits on."
   - The line is about 140 characters of text, over the 120-character prose limit. The reviewer's proposed text was applied verbatim without a break.
   - It splits cleanly at the comma before "since".

**No finding on the other proposals**
- **F2 (generation counter):** I built the interleavings and none keeps a stale answer. The commit happens before `forgetLayouts` runs, and the counter is bumped before `Clear`. A reader that loaded the counter after the bump re-read git after the commit. A reader that loaded it before the bump sees a changed counter and does not store. The failed-read and missing-`.git` paths never store, so they are unaffected.
- **F1 (test):** I checked the three rows against mutations, reading the code and the passing gate rather than re-running them.
  - Dropping `Store` makes row 1 fail, because the re-read finds no marker.
  - Dropping the `forgetLayouts` call in `commitAnchorMarker` makes row 2 fail, because the cached false is returned.
  - Storing a failed read makes row 3 fail.
  - Running it serially is correct. Parallel tests stay paused until serial top-level tests finish, so no other test's `Clear` can race the rows.
- **F4 and F5 (`doc.go` text):** these are consistent with `MergeStateOperation`'s own doc and with fabric's replay running only `git cherry-pick`.

**Tool calls**
No further tool calls. My context already held every changed file.
