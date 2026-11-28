```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function or branch the diff changes, I ran one concrete input through the code step by step.

**Per function or branch: the case I ran, and whether it is correct**

- **`redirectPushes`.** The first call sets `pushRedirect` and later calls keep it. I ran `git init /tmp/r && git -C /tmp/r config url.x.insteadOf y && ln -s a b && git -C /tmp/r push /tmp/r main`. The `config` call sets `rulePushAfterRedirectKey`, the `ln` leaves it in place, and the push returns the key rule. Correct.
- **`runsMutatingGit`, an empty command.** `""` gives one empty segment and returns `("", "")`. Correct.
- **`runsMutatingGit`, two counted invocations in one command.** I ran `git init /tmp/r && git -C /tmp/r push /srv/t.git main && git commit -am x`. The first counted call is the push: `rule` becomes `rulePushToPathOutside` and `pushFrom` becomes `/tmp/r`. The commit then clears `onlyOwnPushes`, so `pushFrom` ends as `""` and `rule` stays the push's rule. This matches the base's `!mutates` handling. Correct.
- **`runsMutatingGit`, the `ln` branch.** I ran the incident command. `ln` sets `rulePushAfterLink`. The `commit` and `remote add` calls run inside `$S/r` and pass. The push returns `$S/r` and the `ln` rule, so the result is `("$S/r", rulePushAfterLink)`. Correct.
- **`gitInvocationMutates`, a help request.** `git commit --help` returns `("", "")`. Correct.
- **`gitInvocationMutates`, the outside branch.** `git -C /tmp/other commit` returns `ruleOutsideCreated`. A commit inside a created repository returns `""`. Correct.
- **`gitInvocationMutates`, a tag or branch that only lists.** Both return `""`. Correct.
- **`gitInvocationMutates`, `config` outside every created repository.** `config` is not in `mutatingGitSubcommands`, so the call itself returns `""` but still sets the redirect rule. This is unchanged from the base. Correct.
- **`pushMutates`, a redirect already set.** It returns `from` and the stored rule. Correct.
- **`pushMutates`, `--repo`, and `-u` from outside.** These return `rulePushWithRepo` and `rulePushSetsUpstreamOutside`. Inside a created repository, `-u` falls through to the destination checks. Correct.
- **`pushMutates`, no operands.** It returns `rulePushNoDestination`. Correct.
- **`pushMutates`, a named remote.** A remote marked as leading in returns `""`; one marked as not leading in returns `rulePushToRemoteOutside`. Correct.
- **`pushMutates`, a bare word.** `origin` returns `rulePushToRemoteOutside`. But the scp-style location `host:repo`, which holds no `/` and no leading `.`, also takes this branch, so it is named as a remote rather than a URL. That is wrong; see F1.
- **`pushMutates`, a path or URL.** `/srv/t.git`, `file:///tmp/r` and `git@h:x/y` return `rulePushToPathOutside`; `../remote.git` inside a created repository returns `""`. Correct.
- **`recordRemote`, an unknown directory.** It sets `rulePushAfterUnknownRemoteChange`, or keeps an earlier rule. Correct.
- **`auditClusterRound`.** A nil audit still returns `ErrClusterForksMissing` with no way forward, so the retry classification in `shedadapters/burlerreview.go:462` is unchanged. A violation is wrapped with `%w`, which keeps `*forkViolationError` in the chain; no caller matches it, and every caller treats it as a hard failure, as before. Correct.
- **`forkViolation`, the message.** For `git commit -am wip` it reads `…("git commit -am wip"), counted as a state-changing git command outside every repository the fork created — …`. Correct.
- **`violationWayForward`, the incident on the review path.** `targetUntouched` is true and `pushFrom` is `$S/r`, so the "no target change was found" text is returned. That text claims no fork wrote a file or ran a state-changing git command in the target's working tree. The incident fork ran `mkdir`, `git init`, `git commit` and `ln` under `/wt/.scratch/exp2`, a path lexically inside the worktree, and the audit sees neither Bash file writes nor where the target is. The claim is stronger than the audit can know; see F2.
- **`violationWayForward`, a named spawn on the review path.** It returns "no target change … resume the run". Correct, within what the audit counts.
- **`auditFixRound`.** Only the function name changed; the behaviour is the same as the base. Correct.

**Findings**

- **F1. LOW, design.** Location: `internal/burlerengine/cluster.go:337-340` (`pushMutates`). An scp-style destination with no `/` and no leading `.`, such as `host:repo`, fails `looksLikePath`. It is therefore reported under `rulePushToRemoteOutside`, "a push to a remote the fork did not point…", instead of `rulePushToPathOutside`, "a push to a URL or path…". The cause is that `isGitURL` is checked only after the `!looksLikePath` early return; a destination such as `host:a/b` reaches `isGitURL` and is named correctly. The command is still counted, so this is a wrong label, not a missed violation. Fix: test `isGitURL(destination)` before the `looksLikePath` branch, and add a row such as `git -C /tmp/r push host:repo main` → `rulePushToPathOutside` to `TestMutatingGitCommandsNameTheBrokenRule`.
- **F2. LOW, design.** Location: `internal/burlerengine/cluster.go:941` (`violationWayForward`, untouched-push branch), now reached from the review path at `cluster.go:880`. The text says "no fork wrote a file or ran a state-changing git command in the target's working tree". `targetUntouched` only counts Write and Edit tool calls and git invocations outside created repositories. It cannot see Bash writes such as `ln`, `mkdir` or redirects, and it does not know where the target lies. In the incident, a commit, an `ln` and a `git init` all ran under `/wt/.scratch/exp2`, inside the worktree path. The regression row at `cluster_test.go:313-326` pins this claim for exactly that case. Fix: word the clause as what the audit found, for example "the audit found no write or edit call and no state-changing git command outside a repository the fork created". Alternatively, name the limit in the way forward.

**Tool calls:** none; I made no tool calls beyond the context I inherited.
