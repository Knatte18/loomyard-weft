- F1, F2, F5, F6: the searches find no remaining member (the literal `/proc/self/cgroup`, a 10 s bound, `cat` in the start script, a `cgroup.kill` writer other than `killScope`). No findings.
- F3 (wording that promises the kill): **LOW**: `internal/proc/doc.go:54` says "The call reports that kill and then kills what the command left in the scope through its `cgroup.kill`."
  After the fix the scope is also killed when a child was OOM-killed and the command was not SIGKILLed, where the call reports no kill, so the sentence no longer matches.

Findings left unchecked: F4 and F7 are test-coverage findings rather than set findings.

Tool calls:
1. One Bash call running all five greps, because the context lacked the rest of the repository, including `doc.go`.
