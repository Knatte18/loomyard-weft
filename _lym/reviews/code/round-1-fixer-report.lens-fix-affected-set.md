```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

I checked three sets, one search each:

- **F1:** every non-test use of `dirs.Anchor` in `codexengine`. The user-setup request was the only consumer that took the unresolved path next to a resolved `RepoRoot`. The remaining uses are the sandbox probe's working directory and the readers of the checkout at `home.go:241` and `codexengine.go:312`, which resolve it themselves or need no resolution.
- **F5:** the only place a directory name is skipped is `unreadScanSkippedDirectories` at `instructionfiles.go:34`. The git-dir check at `home.go:91` is unrelated.
- **F1 callers:** `UserSetupRequest.WorkDir` is read only by the claude user-setup source, which `usersetup.go:19` documents as the session's working directory. No other consumer was left on the old value.

Left unchecked: F8's "contained tests" set. The redundancy report isn't in my context, so I couldn't rerun its search. I also didn't check other docs that might still describe the codex launch as a prompt file (F2). I reached the call cap on F1, F5 and the `WorkDir` consumers first.

Tool calls:
1. One Bash call with three greps (`dirs.Anchor`, skipped directory names, `WorkDir`), because my context didn't hold the uses of the raw anchor outside `Prepare`.
