```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens review of the functions and branches this diff adds or changes.

**List: each function or branch, the case run, and the result**

- `cloneDestination`, empty `into`: returns `cwd`. Correct.
- `cloneDestination`, absolute `/a/../b`: returns `/b`. Correct, same as the old inline switch.
- `cloneDestination`, relative `x`: returns `cwd/x`. Correct, and stays absolute because `CwdFrom` always carries an absolute path.
- `runCloneWithReset`: it passes `cloneDestination(cwd, into)` where it used to pass `dest`. Behaviour is unchanged, and the usage check still runs before the destination is computed. Correct.
- `verbTargetDir`, `CwdFrom` fails: the error comes back, `sandboxedRoleRefusal` skips the pass and refuses. This fails closed. Correct.
- `verbTargetDir`, verb other than `clone`: returns cwd. Every topology and weft verb resolves its hub from that cwd through `resolveWarpLocation` → `lymcwd.Resolve`, so the judged directory matches the one the verb acts in. Correct.
- `verbTargetDir`, `clone --into &lt;dir&gt;`: returns `cloneDestination(cwd, into)`, the same value `runCloneWithReset` uses. Correct as far as the string goes.
- `inOwnSandboxRoot`, existing `&lt;h&gt;/slug/.lym/harden-sandbox/hub-1`: the walk reaches `&lt;h&gt;/slug/.lym/harden-sandbox`, `ResolveWorktree` gives name `slug` and path `&lt;h&gt;/slug`, and the result is true. Correct.
- `inOwnSandboxRoot`, `..` escape (`&lt;sandbox&gt;/../../../other`), existing or not: `EvalSymlinks` or `Clean` collapses it to `&lt;h&gt;/other`, so the result is false. Correct.
- `inOwnSandboxRoot`, `dir="/"`: no match and the parent equals current, so the result is false with no infinite loop. Correct.
- `inOwnSandboxRoot`, `ResolveWorktree` fails (no `.git`, or the recorded anchor is unreadable): false. Fails closed. Correct.
- `inOwnSandboxRoot`, `&lt;own&gt;/sub/.lym/harden-sandbox` (sub is not a worktree root): `FindRoot` climbs to `&lt;own&gt;`, so `WorktreePath()` ≠ `&lt;own&gt;/sub` and the result is false. Correct.
- `inOwnSandboxRoot`, existing symlink inside the sandbox pointing at the real hub's prime: `EvalSymlinks` lands outside, so the verb is refused. Correct.
- `inOwnSandboxRoot`, a dir inside the own sandbox under a nested `.lym/harden-sandbox` of a sandbox worktree with another name: the nearest root wins, so the call is refused. This only fails closed; it widens nothing. Correct.
- Guard passes for a cwd in a sandbox hub container but outside any sandbox worktree (also the sandbox root itself): the verb's `lymcwd.Resolve` climbs to the real worktree `&lt;own&gt;` and the strict cwd gate refuses, so nothing acts on the real hub. Correct.
- `inOwnSandboxRoot`, a destination that does not exist yet, reached through a symlinked ancestor: **wrong**. See F1.
- `sandboxedRoleRefusal`, placement of the new pass: it sits after the teardown-child pass, the bypass-flag refusal and the reader/merge-task pass. A bypass flag on a non-push verb is still refused before the sandbox pass is reached, so the pass widens neither branch above it. Correct.
- `sandboxedRoleRefusal`, refusal text: it still contains `from the task session` and now names `.lym/harden-sandbox`. Correct.

**Findings**

1. MEDIUM, design. Location: `internal/fabriccli/roleguard.go:45-48` (`inOwnSandboxRoot`), together with `roleguard.go:35-36` (`verbTargetDir` for clone).
   Summary: when the destination does not exist, the guard compares it as written, with no symlinks resolved. The guard and the clone then judge different directories. This fails in two directions:
   - **False refusal.** When an ancestor such as the worktree path, the hub or `$HOME` is a symlink, `worktree` stays unresolved, but `lymcwd.ResolveWorktree` resolves symlinks before `FindRoot` (`internal/lymcwd/lymcwd.go:202-211`). `location.WorktreePath()` is then the resolved path and never equals `worktree` (`roleguard.go:53`), so a legitimate `clone --into &lt;own sandbox&gt;/hub-2` is refused.
     - A symlinked cwd reaches this check because Go's `os.Getwd` returns `$PWD` when it names the same directory.
     - The test row `IntoOwnSandboxRootFromItsWorktree` (`roleguard_test.go:286`) would fail wherever `t.TempDir()` sits under a symlink, such as macOS `/var` → `/private/var`.
   - **Pass, then a write outside the sandbox.** Take `clone --into &lt;own&gt;/.lym/harden-sandbox/link/new`, where `link` points outside the sandbox. `Clean` keeps the lexical path under the sandbox root, so the guard passes. `CloneHub` then builds `hubPath = HubPath(dest, name)` (`internal/fabricengine/clone.go:209`) at the link's target, and with `--reset` it tears down an existing hub there (`clone.go:211-212`).

   This contradicts the doc's "a link counts where it lands" (`roleguard.go:42`), which holds only for a destination that already exists.
   Fix: resolve the deepest existing ancestor of `dir` with `EvalSymlinks`, then re-join the parts that do not exist yet before walking.
   Evidence: `roleguard.go:45-53`, `lymcwd.go:202-211`, `fabricengine/clone.go:209-212`.

No other correctness findings: every other branch in the list ran correctly, and every failure path fails closed.

**Tool calls**
- `grep` of `func CloneHub` in `internal/fabricengine`: the manifest lacked how `CloneHub` derives the hub path from the destination and what `--reset` does there, which F1's severity depends on.
