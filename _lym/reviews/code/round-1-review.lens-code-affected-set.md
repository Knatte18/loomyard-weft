```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. For each change, I list what it forces outside the lines the diff touches, and whether the diff covers it.

**Per-change list**

1. **New `PATTERN-remote-refs-lock` entry (`PATTERN.md:30`).**
   - Set found: the entry text, `pattern/` background files, and every non-test fetch, pull and push site.
   - Covered: the `FetchWithArgs` conversions in `fabricengine` (`add.go`, `checkout.go`, `weftbranch.go`, `cleanupremotewarp.go`, `remoteonly.go`, `remove.go`, `taskbranchdiscard.go`, `archive.go:160`) and the `Fetch`, `Pull` and push methods in `gitrepo`.
   - Exempt as the entry says: the `--refmap=` fetches in `remoteleftover.go:110` and `:140`.
   - Not covered: `archive.go:128` still runs a raw `gitexec.Run push` of a tag, outside the lock, with no `Pattern-Departure:` trailer. The entry says "pushing to origin ... through `gitrepo`'s ... push methods", so it contradicts the code. A tag push writes no tracking ref, but the entry only exempts `--refmap=` fetches. (F1)
   - `recordsprobe.go:180` runs a raw `fetch --depth 1 origin` into a private temp clone. It is harmless, but the entry's wording does not exempt it either. (Also F1.)
   - Not covered: no `pattern/` background file and no `(test)` marker. The entry names no enforcing test, so a raw `fetch` can be reintroduced unnoticed. `PATTERN-parent-lock` and others also lack both, so this is not clearly a rule violation. (F2)

2. **`PushRebaseFree` doc (`internal/gitrepo/push.go:153`).**
   - Set found: the `PushRebaseFree` doc comment and the `PushCoalesced` doc.
   - The doc still ends "Lock-free." The method now takes the remote-refs lock on each attempt. "Lock-free" may mean "no push lock", but it now reads as contradicting the file header and `doc.go`. (F3)

3. **`FetchWithArgs` added to the pinned run-bound method list.**
   - Set found: `cmd/lym/gitrepoboundary_test.go`, the `doc.go` method list, and `PATTERN-gitrepo-client-boundary.md`.
   - Covered: the pinned list and the `doc.go` list both add `FetchWithArgs`.
   - Note: the scan comment "10 non-test .go files today" (about line 95) is now stale. `remoterefslock.go` is an 11th file. The floor is 5, so it is harmless and I raise nothing.

4. **`judgeAgainstOrigin` and the lock held through the verdicts.**
   - Set found: its callees (`lsRemotePair`, `fetchMovedHeads`, `judgeBranch`), plus `originTargets` and the `originCallHook` tests.
   - Covered: `judgeBranch` calls only `gitrepo.New(repoDir)` for reads, with no `Fetch`, `Pull` or `Push`. `fetchMovedHeads` and `lsRemote*` use `gitexec` directly, so the non-reentrant lock cannot deadlock on itself.
   - Covered: the `fetchMovedHeads` and `ReconcileOrigin` doc comments, and `fabricengine/doc.go:70`, match the code.

5. **Fetch sites moved from `gitexec.Run` to `FetchWithArgs`.**
   - Set found: callers that classify the error (all wrap with `%w`; none inspect the type), the `remoteonly.go` receiver (`f.code` replaces `f.warpPath`), and the unused `gitexec` imports.
   - Covered: `remoteonly.go` drops the import and the other files still use `gitexec` elsewhere.
   - Not checked: whether `f.code` is rooted at `f.warpPath`. I read the diff but did not verify the constructor, so I raise no finding.

6. **`Pull`, `Fetch` and the push methods wrapped in `withRemoteRefs`.**
   - Set found: the `//gitexec:raw` markers on `Pull` and `Fetch`, and `pull_test.go` and `fetch_integration_test.go` (the no-`fatal:` surface).
   - Covered: the markers remain on the `r.run` lines inside the closures, and the error text is unchanged.

**Findings**

- **F1 — MEDIUM, consistency.**
  - Location: `PATTERN.md:30` against `internal/fabricengine/archive.go:128`, with `recordsprobe.go:180` as the second site.
  - Summary: the entry says every push to origin goes through `gitrepo`'s methods under the lock, yet the archive tag push is a raw `gitexec.Run push` outside it. The entry's only exemption is `--refmap=` fetches, and no `Pattern-Departure:` trailer explains the gap. Either narrow the entry to ref-writing calls and say so, or route the tag push through `gitrepo`.
  - Evidence: `archive.go:127-130`, `PATTERN.md:30`.

- **F2 — LOW, consistency.**
  - Location: `PATTERN.md:30`.
  - Summary: the new entry has no `(test)` marker and no enforcing test, and no `pattern/PATTERN-remote-refs-lock.md` background. The `gitrepoboundary_test.go` change pins only that `FetchWithArgs` calls `runChecked`, not that fabric fetches go through `gitrepo`.
  - Evidence: `PATTERN.md:30` against entries such as `PATTERN-gitrepo-client-boundary` that carry both.

- **F3 — LOW, consistency.**
  - Location: `internal/gitrepo/push.go:153`.
  - Summary: the `PushRebaseFree` doc says "Lock-free." but each attempt now takes the remote-refs lock. Reword it to "Takes no rebase or push-serialising lock beyond the remote-refs lock", or drop the line.
  - Evidence: `push.go:147-154` against `push.go:138-143`.

**Tool calls**
1. `sed -n 522,640p internal/fabricengine/originreconcile.go | grep ...` — checked that `judgeBranch` and its body call no `gitrepo` fetch, pull or push, which my context lacked.
