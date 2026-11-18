```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the F1–F7 change set. No list was required.

**LOW — stale comment on the holder's redirect.**
- Changed text: the `scopeStartScript` doc line "dash keeps the shell's stdout on a background builtin whose own redirection says otherwise" in `internal/proc/memorycap_linux.go`.
- The holder is no longer a background builtin (`read -r line <&4 &`). It is now the external command `setsid sh -c 'read -r line' <&4 &`.
- The sentence describes a case the script no longer has, though the subshell's `/dev/null` redirects still do their job.
- A reader would be misled about why the redirects are there.

**LOW — a missing `setsid` ends the holder at once and nothing fails.**
- Changed text: `setsid sh -c 'read -r line' <&4 &` in `scopeStartScript`.
- Without `setsid` the background job exits 127, which no one reads. The holder ends before the command does, and the `memory.events` race the diff closed comes back.
- F4's test only polls for scope removal, so it passes on such a host. The same holds for F7, which sees no child.
- `setsid` ships with util-linux on systemd hosts, so this is unlikely. The holder start is an unchecked assumption all the same.

**LOW — F7 test fails on a kernel without `/proc/<pid>/task/<tid>/children`.**
- Changed text: `TestRunMemoryCapped_TheHolderIsNeverAChildOfTheCommand`.
- `CONFIG_PROC_CHILDREN` is off on some kernels. On those, `cat` exits 1, so `err != nil` and the test fails with no holder defect present.
- The test skips only on `MemoryCapCoversTree`.

Traced and found correct:
- **F1:** `cgroupMemoryMaxFile` now sits in `memorylimit.go`, which builds on every platform. `memorycap_linux.go` and `readMemoryLevel` both use it, so there is one declaration and no literal left in non-test code.
- **F5 script:**
  - The subshell is a foreground job and is waited on.
  - The `setsid` job is not a process group leader, so `setsid` execs directly. If it did fork, the child would still hold fd 4.
  - The explicit `<&4` overrides the `/dev/null` stdin.
  - The holder reparents out of the shell, so it is never the shell's child.
- **F2:**
  - The head-bounded command exceeds the 64 MB cap, so `tail` is killed and the holder keeps the scope readable, which makes the `sleep` unnecessary.
  - The fixed size means a cap that fails to hold costs 256 MB at most.
- **F6:**
  - `testMemoryCap + 1` rounds down to `wantMax = 128 MB`, which equals the kernel's page-rounded `memory.max`, so the run passes.
  - With the round-down removed, the script compares `134217729` with `134217728` and exits 125. That returns an `ErrMemoryCap` error, so `ended` is true against `wantEnded` false and the row fails.
  - `limit` is `int64`, and the untyped constants convert. The non-scope and Windows paths accept the same cap.
- **F4:**
  - With `release.Close` dropped, the holder stays up until a GC finalizer closes the pipe. The scope then outlives the one-minute poll in all but unlikely runs, so the test fails with the defect in place.
  - The `0::` parse matches what `cat /proc/self/cgroup` prints inside the scope.
- **F7:** with a plain `&` child of the shell, the shell's pid survives each `exec`, so `cat` would list that child and the assertion would fail.

No tool calls made: my context already held every changed file.
