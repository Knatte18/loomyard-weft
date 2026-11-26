```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fixer's change set (F1–F9).

**Findings**

1. **MEDIUM**: the F5 rewording in `internal/loomcli/prepush.go` (file header, `prePushCheck` doc, `guardedCommit` doc) names the wrong guarded sites.
   - The new text says the check guards "the landing's commit and push", and that `guardedCommit` is "for the landing's commit", with "the landing's push after the commit" reading its stop.
   - The only `guardedCommit` caller is `pullRequestRecordCommitter` (`landingdeps.go:530`). It commits PR-Review's pull-request record and pushes it through `guardedPush` (`landingdeps.go:534`). It is also built from `prreviewgoto.go:80`, `prfindings.go:344` and `prcontinue.go:206`.
   - The other `guardedPush` caller is `PushTask` (`landingdeps.go:423`). My grep found no landing commit or push behind the check.
   - The comments should name the pull-request record's commit and push, PR-Review's task push and the pusher. The review's proposed wording carried this error.

2. **MEDIUM**: F1's pushes-off path contradicts sibling statements that the removal waits for origin to hold the tips.
   - `pattern/PATTERN-pair-remote.md:94-95`: "Worktree-Teardown pushes the pair to completion with the prime lock released, and removes it only while origin's tracking refs hold both tips."
   - `internal/battenshed/doc.go:111`: "the second hold removes the pair once origin's tracking refs hold both its tips."
   - With `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` set, `teardownPushSeams` pushes nothing, reads level and the pair is removed. None of these texts states that exception. The `TeardownDeps.Push` and `Level` docs in `deps.go:375-386` do not state it either.

3. **LOW**: F1's three-push bound is missing from the twins of the `teardown.go` doc.
   - `internal/battenshed/doc.go:112`: "A tip that moved since the push releases the second hold and pushes again."
   - `internal/battenshed/deps.go:384`: "A false read means a tip moved since the push, so the teardown pushes again."
   - Neither says that a `Level` still false after `maxTeardownPushRounds` pushes ends Stuck.

4. **LOW**: the edited `DobbyRuns` doc line in `internal/battencli/dobbyreads.go` ("A run this machine owns, with its pair here, first restarts its pair's pusher through startPusher when …") is over the 120-character line limit. It needs a semantic break.

`internal/beamcli/doc.go:210,226` still holds with the new `hubDobbyDeps` wrapper.

**Tool calls**
1. Grep over battenshed, PATTERN files and runpusher docs for teardown-push statements; my context lacked these files.
2. Grep for other pre-push check and `hubDobbyDeps`/`DobbyRuns` references; my context only had `prepush.go` and a caller grep.
3. `sed` of `landingdeps.go` around the guarded call sites; my context had only their line numbers.
4. `sed` of `battenshed/doc.go`, `deps.go`, `PATTERN-pair-remote.md` and `beamcli/doc.go` spans; my context lacked their surrounding text.
