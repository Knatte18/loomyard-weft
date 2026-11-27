```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-shaped findings' searches and check that the fix covers every member.

Findings left unchecked: none of the set-shaped ones I identified (F1, F2, F3, F8, F13, F14, and the `ResolveShellPath` rename the F3 fix introduced).

no findings

Tool calls:
- Tier and spawn references per test file, plus `t.Error`/`t.Fail`/`t.Skip` in the kit's non-test files: my context lacked a post-fix sweep across all the files (F1, F2).
- `resolveShellPath` mentions across Go and markdown: my context only held the call sites found by the Go-only search under `internal` (the F3 rename). None remain outside the excluded `burlerengine` function of the same name.
- Recordings searched for this machine's user, host, home, uid and `/usr`, `/tmp` and `/run/user` paths: my context showed only part of the re-recorded files (F3, F8). No matches.
- Tooling words in the change description, and the kit lists in docs and patterns: my context lacked a search of either (F13, F14). The one hit, "filed for a follow-up", names no tooling the finding listed.
