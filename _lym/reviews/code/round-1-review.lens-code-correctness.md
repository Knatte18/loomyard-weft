```yaml
lens: code-correctness
outcome: reported
```

Task: check the correctness of the code the diff changes (code-correctness lens).

**Per-branch list**

- `ownerPreflight`, found status at Worktree-Teardown with the worktree absent and the owner here: the guard passes, `taskWorktreePresent` returns false, and the log line is written. `refuseOwnedElsewhereOnLocalBoard("")` finds no other owner and returns nil. No reconcile runs and no marker is written. Correct.
- `ownerPreflight`, same state with the local board showing owner "there": returns `OwnedElsewhereError` with text `…owned by "there"; run \`lym adopt X\` from this machine's prime…`. There is no "once origin answers" clause. `passExitCode` still maps it to exit 77. Correct.
- `ownerPreflight`, Teardown with the worktree present: `present` is true, so it falls through to the `Machine == nil` check and then to `reconcileResume`. Correct.
- `ownerPreflight`, `taskWorktreePresent` returns an error (a stat failure other than not-exist, or `ResolveWorktree` failing): the error is returned bare, so the resume is refused. Correct fail-closed behaviour, and the same as the existing Worktree-Create branch. The message is unwrapped and names no way forward. This is a minor wording gap and the same as the existing branch (LOW, consistency, `owner.go:237`).
- `ownerPreflight`, Teardown with the worktree absent and a nil `c.owner.Machine`: `refuseOwnedElsewhereOnLocalBoard` returns nil at its first line, so the run resumes with no reconcile. This is acceptable. Before the diff, a receiver with no seams ran the reconcile, which is a behaviour change. The doc comment's "bar that teardown" covers it. Correct.
- `ownerPreflight`, `found == false`: the guard is skipped and the path is unchanged. Correct.
- `ownerPreflight`, a directory exists at the worktree path but `ResolveWorktree` fails (half-removed or debris): the error is returned and the resume is refused. A debris-only path therefore blocks the teardown retry. The teardown row itself handles half-removed pairs, but this pre-flight stops it from being reached. Compare the `Worktree-Create` branch, which has the same `taskWorktreePresent` behaviour. It is an edge case, arguably an open risk, not a defect.
- `refuseOwnedElsewhereOnLocalBoard(retry)` with an empty or non-empty `retry`: the format reads `"…%srun \`lym adopt…"`. The unreachable caller passes "re-run `lym batten run S` once origin answers, or " and the output is unchanged from the old text. The empty string gives "…; run `lym adopt X`…". Correct. A board read or resolve error returns nil with a Warn. This is unchanged and is the documented unjudgeable-resumes behaviour, and with the skip it now also applies to the teardown path.
- `ownerPreflight` ordering: the `thisMachine` error (an empty hostname) is returned from the new branch with the "fix the hostname, then re-run lym batten run" hint. Correct.
- `finishRemoval` with `RemoteBranchError` set: the message ends "re-run lym batten run <slug> to retry the deletion". `slug` is the function parameter and is in scope. Correct (D4).
- `reconcileResume`, the unreachable path: it passes the retry string with `c.slug` and the marker is still written. Unchanged behaviour. Correct.
- Callers of `ownerPreflight`: `arm.go:922` (the pre-step) and `preflightRunOwner` (`arm.go:735`, `cli.go:452`) both go through the same function, so both get the skip. Correct.
- Under `--continue-pass`, `ownerPreflight` is skipped (per the `battenPreRun` doc). Unchanged.

**Findings**

1. LOW, consistency, `internal/battencli/owner.go:237`: the `taskWorktreePresent` error on the new Teardown branch is returned unwrapped, so the operator sees a bare stat or resolve error with no resume way forward. The existing Worktree-Create branch (`owner.go:256`) has the same shape. I raise it only for symmetry, and it is not a correctness defect.

No BLOCKING or MEDIUM correctness defects found.

**Tool calls**
1. Read `arm.go` lines 905–935, because the manifest did not show the second `ownerPreflight` caller (`arm.go:922`) and I needed to confirm that it reaches the same function.
