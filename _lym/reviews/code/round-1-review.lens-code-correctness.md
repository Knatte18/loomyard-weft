```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function and branch the diff adds or changes against concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `mutatingGitCommands` / `runsMutatingGit`. Case: one command holding a push from `/tmp/r` and then `git commit` in the target. `pushFrom` comes back "" because `onlyOwnPushes` is false. Correct. Case: two pushes from two created repositories. `pushFrom` is the first one. Correct.
- `gitInvocationMutates`, init with no operand. Directory falls back to "." and `record` refuses it. Correct.
- `gitInvocationMutates`, worktree add from a created repo. The worktree is recorded. Correct.
- `gitInvocationMutates`, worktree add from a repo the fork did not create. Nothing is recorded. Correct.
- `pushMutates`, no destination: counts. Correct.
- `pushMutates`, `--repo=` with a value: counts. Correct.
- `pushMutates`, push to a bare remote name no `git remote` call set: counts. Correct.
- `pushMutates`, push from the target (`from` is "") to a path inside a created repo: passes. Wrong for `-u` and `--recurse-submodules` (F8).
- `pushMutates`, relative destination pushed from a subdirectory of a created repo: wrong (F1).
- `recordRemote`, add, then set-url, then push: correct.
- `recordRemote`, `set-url --push` outside, then a plain `set-url` back inside: wrong (F2).
- `recordRemote`, remote re-pointed from a worktree of the created repo: wrong (F3).
- `recordRemote`, rename and remove: both names marked false. Correct.
- `recordClone`, source `src.git` with no directory: records `src`. Correct.
- `recordClone`, source `/`: base is ".", nothing recorded. Correct.
- `recordClone`, `--origin=up`: remote `up`. Correct.
- `recordClone`, source `remote.git`: wrong, over-flags (F5).
- `recordClone`, clone that fails and leaves no directory: the root is still recorded (folded into F7).
- `cloneDirectoryName`: `"/tmp/r/"` gives `r`, `"a/.git"` gives `a`, `"host:x/y.git"` gives `y`. Correct.
- `cloneDirectoryName`, `--bare` clone: returns `r` where git makes `r.git`. Errs toward counting the command, so acceptable.
- `isGitURL`: `file:///x` is a URL, `./a:b` is a path, `foo:bar` is a URL. Correct, and errs toward counting.
- `namesPushRedirectKey`: case-insensitive match of `url.X.insteadOf=` is correct. Over-matching on operands is F5.
- `parseGitInvocation`, `-c remote.pushDefault=x`: flagged. Correct.
- `parseGitInvocation`, `--config-env=url.x.insteadOf=E`: not flagged. Wrong (F6).
- `gitOperands`: `--` followed by several words returns all of them. Empty args returns nil. Correct.
- `rootContaining`, nested roots `/tmp/r` and `/tmp/r/w`: innermost wins. Correct.
- `rootContaining`, root `"/"`: matches only `"/"`. Errs toward counting.
- `record` with an anchored ancestor of the target: accepted (F7).
- `forkViolation` / `fixViolationWayForward`, a write plus a push: revert. Correct.
- `forkViolation` / `fixViolationWayForward`, a target commit in fork a plus a push from a created repo in fork b: revert. Correct.
- `forkViolation` / `fixViolationWayForward`, a nested spawn plus another fork's push from a created repo to an unknown remote: wrong (F4).
- `forkViolation` / `fixViolationWayForward`, a named spawn alone: "no change, resume". Correct.
- `archiveStaleOutput`, absent file: returns "" and nil. Correct.
- `archiveStaleOutput`, stat error: returned. Correct.
- `archiveStaleOutput`, rename error: returned. Correct.
- `archiveStaleOutput`, name collision: suffix added. Correct.
- `archiveStaleOutputs`: stops at the first error, as before. Correct.
- `archiveReport`, archive error: logged, no record. Correct.
- `archiveReport`, no report present: no record. Correct.
- `archiveReport`, cancelled context: records the cancellation reason. Correct.
- `recordRerun`, acceptance read error: logged. Correct.
- `recordRerun`, snapshot write error: logged. Correct.
- `recordedRerun`, record for another digest: zero value. Correct.
- `recordedRerun`, read error: zero value, logged. Correct.
- `Call` rerun switch, attempt 1 finds a leftover report: reason set. Correct.
- `Call` rerun switch, attempt 2 with an archived report: reason names the prior attempt. Correct.
- `Call` rerun switch, attempt 2 with no report while an older record exists: wrong, stale reason (F9).
- `fixWork` / `fixRerunStatement`: zero rerun returns the base text, an empty `Report` takes the "without a fixer-report" branch, and both branches render. Correct.

**Findings**

**F1 — MEDIUM, design. `internal/burlerengine/cluster.go:270-271, 324, 386-391`**
A relative push destination or remote URL is resolved against the directory the invocation ran in. Git resolves it from the repository's top level, because git changes to the top level before the transport resolves a local path. That lets a push into the target pass.
- Example: `git init r && mkdir r/x && git -C r/x push .. HEAD:refs/heads/evil`.
- The audit resolves `r/x` plus `..` to `r`, a created root, so the push passes.
- Git resolves `..` from `<start>/r`, which is the target worktree, and pushes a branch into the target repository.
- The same happens through `git -C r/x remote add origin ..` followed by `git -C r push origin`.
- Evidence: `leadsInto` uses `invocation.dir` (cluster.go:390), and `recordRemote` computes `leadsIn` the same way (cluster.go:324).

**F2 — MEDIUM, design. `internal/burlerengine/cluster.go:320-328`**
A plain `git remote set-url origin <inside>` marks `origin` as leading in. It only replaces the fetch URL, so a push URL set outside by an earlier `set-url --push` still applies.
- Sequence: `remote add origin /tmp/b.git`, then `remote set-url --push origin /srv/target.git`, then `remote set-url origin /tmp/b.git`.
- After it, `push origin` passes while git pushes to `/srv/target.git`.
- `--push` is never told apart from a plain set-url.

**F3 — MEDIUM, design. `internal/burlerengine/cluster.go:165, 313-328, 265`**
Remote state is keyed per created root, but a worktree added from a created repo shares that repo's config.
- Sequence: `git -C /tmp/r remote add origin /tmp/b.git` (recorded as leading in under `/tmp/r`), then `git -C /tmp/r worktree add /tmp/w`, then `git -C /tmp/w remote set-url origin /srv/target.git` (recorded under `/tmp/w`).
- After it, `git -C /tmp/r push origin` still reads `true` under `/tmp/r` and passes, while git pushes to `/srv/target.git`.

**F4 — MEDIUM, design. `internal/burlerengine/cluster.go:732-741, 769-772`**
`targetUntouched` treats a push from a created repo as untouched. When a non-git violation fires first, `pushFrom` stays "", and the halt states something false.
- Example: fork a made a nested spawn. Fork b ran `git init /tmp/r && git -C /tmp/r push origin main`, a push to a remote nothing pointed.
- The halt then says "no fork wrote a file or ran a state-changing git command outside a repository it created; way forward: resume the run".
- That is false, and it drops the "check that the push's destination is none of the target's remotes" step.

**F5 — LOW, design. `internal/burlerengine/cluster.go:210, 403-415`**
For `git clone`, `namesPushRedirectKey` scans every argument, the source and the directory included. `git clone remote.git work` (also a `branch.x` or `url.x` name) sets `pushRedirected`. Every later push of the fork then counts, so the issue's scratch probe still halts when it clones its bare `remote.git`.

**F6 — LOW, design. `internal/burlerengine/cluster.go:451-455`**
Git documents `--config-env` in the attached form `--config-env=<name>=<envvar>`. `gitValuedGlobalOptions[args[i]]` never matches that whole word, so `git --config-env=url.x.insteadOf=E push /tmp/b.git` passes unflagged. A `-c include.path=<file>` or `git config include.path`, which can pull in a redirecting `url.`/`remote.` key, is not flagged either.

**F7 — LOW, design. `internal/burlerengine/cluster.go:352-361, 386-391`**
`record` refuses only relative paths at or above the starting directory. An anchored `git init /home/u` (or `git init <abs target>`, which reinitializes) records a root that contains the target. A path push into the target, such as `git -C /tmp/r push /home/u/.../worktree HEAD:refs/heads/x`, then passes.
- The commit gap predates this diff; the diff extends it to pushes and to clones.
- A clone that fails also leaves its root recorded.
- A later `mkdir c && git -C c commit` then lets git find the enclosing target repo while the audit passes it.

**F8 — LOW, design. `internal/burlerengine/cluster.go:250-272`**
A push from outside any created repo (`from` is "") to a path into one passes, even with options that change state beyond the destination:
- `-u` / `--set-upstream` writes `branch.<b>.remote` into the target's own config.
- `--recurse-submodules=on-demand` pushes submodules to their own remotes.

**F9 — LOW, design. `internal/shedadapters/burlerfix.go:287-294, 269, 448-449`**
- Attempt 2 with no archived report keeps a rerun read from an earlier call's record (`rerun.Reason != ""`). The attempt-2 fixer is then told an earlier call's halt reason and report, never that attempt 1 just died.
- A live fixer that was resumed and died without a report falls through to attempt 1 with no rerun. That fixer is not told an earlier session worked the review, though the target may hold that session's edits.

**Tool calls**
1. `grep` of `highestCompleteRound` and `cancelErr` in `internal/shedadapters`: to judge when attempt 1 can find a leftover report and whether `ctx.Err()` matches `cancelErr`. Neither was in the manifest.
