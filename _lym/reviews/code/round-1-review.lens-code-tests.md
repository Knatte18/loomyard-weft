```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them**

*cluster.go: the fork git guard*
- Push to a path inside a fork-created repository passes: `TestFirstMutatingGitCommandScopesToForkRepositories` row "push to a path inside a created repository". Every row uses an absolute destination; no row pushes to a relative path such as `../remote.git`.
- Push through a remote `git remote add` pointed at a relative created repository passes: row "scratch push probe between a bare and a work repository".
- `git clone` records its repository, so a commit inside the clone passes: row "commit in a clone of an uncreated repository".
- The clone's `origin` leads in only when its source is a created repository: rows "push to the origin of a clone of a created repository" and "push to the origin of a clone of an uncreated repository".
- The directory git derives from the clone source (`cloneDirectoryName`): row "push from a clone in its derived directory".
- `--origin <name>` as a separate word: row "push to a clone's remote named by --origin". The attached `--origin=<name>` branch of `recordClone` and the short `-o` form: none.
- `git remote add` or `set-url` pointing outside: rows "push to a remote pointed outside" and "push to a remote whose URL was set outside".
- `set-url --add`: row "push to a remote given a second URL outside". `set-url --delete`: none.
- `git remote rename`: row "push to a renamed remote". `git remote remove` / `rm`: none.
- Bare remote name the fork never pointed, no destination, `--repo`, URL destination: one row each.
- Redirect through a `git config url.` key and a global `-c remote.` key: one row each. These have none: a `branch.` key, `--config-env`, `git clone -c`/`--config` with a redirect key (the `subcommand == "clone"` arm), and the `--name=value` form in `namesPushRedirectKey`.
- `rootContaining` choosing the innermost of nested created repositories, which keys the `remotes` lookup in `pushMutates`: none. No row creates one repository inside another, so a mutant that returns any containing root survives.
- `runsMutatingGit` setting `onlyOwnPushes`/`pushFrom` across the segments of one command: `TestAuditClusterRound` row "fix: a command that pushes from a fork-made repository and commits in the target keeps the revert way forward".

*cluster.go: the fix-round halt text*
- `forkViolation` computing `targetUntouched` across every fork, not only the violating one: row "fix: a nested spawn beside a fork's commit in the target keeps the revert way forward".
- `fixViolationWayForward`, its three texts: the push text from the row "fix: a push from a fork-made repository …"; the no-change text from "fix: a named fork spawn is the twill fixer violation"; the revert text from "fix: a write call …" and the two rows above. The suffix check now uses `HasSuffix` on the whole way forward.
- The review half's changed message ("… outside a repository it created"): the `TestAuditClusterRound` row "git-mutating bash command in fork" asserts only a substring, which still matches.

*prompt.go / fix.go / profile.go*
- `fixWork` adds the rerun paragraph, with and without an archived report, and omits it on a first fix: `TestFixWork_Rerun`, three rows. No row combines it with the gate-findings or decisions-routed forms; that is plain concatenation.
- `Engine.Fix` passes `opts.Rerun` into the step: `TestEngine_Fix_TableShapeAndFiles`. Its expected step comes from `fixWork(..., opts.Rerun)`, but the wording is pinned independently by `TestFixWork_Rerun`.

*shedadapters*
- `archiveStaleOutput` returns the archived path, or `""` when the file is absent: `TestBurlerFixProducer_Call_TellsARerunFixerWhy` (the stamped Report, and the "left no report" row). `archiveStaleOutputs` callers: `TestArchiveStaleOutputs` and the wider suite, per the reach report.
- A failure exit records "the round halted: <err>" and the next call reads it back: row "a call after a halted fix names the archived report and the halt".
- An attempt-2 retry after a died attempt, with a report: row "a retry after a died attempt that wrote its report names that report". Without one: row "a retry after a died attempt that left no report gives the reason alone".
- A record under another review's digest is not told: row "a rerun recorded for another review is not told".
- Attempt 1 archiving a leftover report and giving the reason "that session never finished the round" (the #176 scenario itself): **none**.
- `archiveReport` replacing the reason with the cancellation when ctx is done (burlerfix.go:144-145): **none**. `TestBurlerFixProducer_Call_Cancellation` checks only the archive count. This is mutation survivor :145.
- The pre-attempt archive persisting its reason via `recordRerun` (burlerfix.go:295-296), for a later call to read: **none**. Mutation survivors :295 (negated condition) and :296 (dropped statement).
- The attempt-2 branch when a recorded rerun is already set and no report was archived (it keeps the recorded rerun): none.
- `fixer_rerun` key in the round snapshot: covered through `updateRoundSnapshot` / `readRoundSnapshot` in the test above.

**Findings**

1. **MEDIUM**, class `design`, at `internal/shedadapters/burlerfix.go:287-289`. Nothing tests the attempt-1 branch that archives a leftover fixer report and tells the fixer "that session never finished the round", with that report. This is the exact #176 scenario. Evidence: grep finds no test asserting "never finished the round"; every `TellsARerunFixerWhy` row starts with no leftover report (burlerfix_test.go:535-573).
2. **MEDIUM**, class `design`, at `internal/shedadapters/burlerfix.go:295-296`. No test reads, from a later `Call`, the record the pre-attempt archive writes. Both mutants survive: the negated condition and the dropped `recordRerun`. The missing row: attempt 1 archives a leftover report, the fixer then ends without a failure exit and without writing a report, and the next `Call` must be told that reason and that report. Evidence: mutation report lines for :295 and :296; burlerfix_test.go:535-573.
3. **LOW**, class `design`, at `internal/shedadapters/burlerfix.go:144-145`. No test pins the cancellation reason "the run was cancelled while the fixer ran". `TestBurlerFixProducer_Call_Cancellation`'s row "during a failing attempt the report is archived" (burlerfix_test.go:627-640) asserts only the archive count; it should also assert the snapshot's `FixerRerun.Reason`. Evidence: mutation survivor :145; grep finds the string in no test.
4. **LOW**, class `design`, at `internal/burlerengine/cluster.go:373-382` (`rootContaining`). No row nests one created repository inside another, so choosing the innermost root, which keys the `remotes` lookup in `pushMutates` (cluster.go:265), is unpinned. Missing row: `git init /tmp/r && git clone /tmp/b.git /tmp/r/sub` with a push of `origin` from `/tmp/r/sub`. Evidence: rows at cluster_test.go:668-688 (diff 001 lines 668-688).
5. **LOW**, class `design`, at `internal/burlerengine/cluster.go:210, 292-297, 319-333, 403-416`. Several branches the diff adds have no row: redirect through a `branch.` key, `--config-env`, `git clone -c`/`--config`, and a `--name=value` key word; the attached `--origin=<name>`; `git remote remove`/`rm`; `set-url --delete`. Each fits as a row in `TestFirstMutatingGitCommandScopesToForkRepositories`. Evidence: diff 001 lines 668-688.

Tool calls:
- grep of `cluster_test.go` for the fix rows and git rows, and of the test files for the rerun reason strings: my context held only the diff hunks of `cluster_test.go` and none of the rerun-reason assertions.
