```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes with concrete inputs.

**Per function and branch: the case I ran, and whether it is correct**

1. `rootProcesses` (`internal/hardenshed/reap.go:197-200`), root resolution. I ran three roots:
   - **A root that does not exist.** `EvalSymlinks` fails with ENOENT, so `roots` holds only `[deps.Root]`. That is the merge base's behaviour. Correct.
   - **A root with no link on its path, written clean and absolute.** `resolved == deps.Root`, so nothing is appended. Correct.
   - **A root written with a trailing separator.** `EvalSymlinks` returns the cleaned path, so a harmless duplicate form is appended. Correct.
2. `rootProcesses`, the resolved root under a `.lym` link. I used `Root=/w/.lym/harden-sandbox` with `.lym` pointing to `/t`, so `resolved=/t/harden-sandbox`.
   - **Cwd `/t/harden-sandbox/hub-one`.** The written root fails `strings.Index`; the resolved root matches at a separator boundary. The process is listed. Correct.
   - **Cwd `/t/harden-sandbox`, the root itself.** `end == len(text)`, so it matches. Correct.
   - **Cwd `/t/other`, a sibling under the link target.** It matches neither form, so the shared link target is not over-matched. Correct.
   - **Cwd `/t/harden-sandbox-old`.** The character after the root is `-`, not a separator, so it does not match. Correct.
3. `rootProcesses` when `EvalSymlinks` fails for a reason other than ENOENT, such as EACCES or ELOOP. The error is dropped and only the written root is compared. This is not a defect: `Reap` calls `ListSandbox(deps.Root)` first, and that walks the same path components, so a failure confined to `EvalSymlinks` with a readable root is not reachable in practice. Correct.
4. `rootProcesses`, an argv or exe naming the resolved path. `namesRoot` runs over Cwd, Exe and Argv for each root, so a resolved-path argv now matches too. That is consistent with the new doc comment. Correct.
5. `rootProcesses` with a deleted cwd, `/t/harden-sandbox/hub-one (deleted)`. The character after the root is a separator, so it matches. Correct.
6. `rootProcesses` with a nil `ListProcesses`. It returns early, before `EvalSymlinks`, so there is no added syscall. Correct.
7. The loop closure in `slices.ContainsFunc(roots, func(root) { namesRoot(process, root) })`. It captures `process` per iteration under Go 1.22+ loop semantics and runs synchronously. The `err` shadowing in the `if` is local and harmless. Correct.
8. The caller and its ancestors. The `protected` check still runs before the root match, so a protected pid whose cwd is under the resolved root stays spared. Correct.
9. `TestReap/EndsAnOrphanWhoseCwdIsUnderTheRootsLinkTarget`, step by step:
   - `ListSandbox` on a root with no hub list yields no hubs.
   - `rootProcesses` lists pid 12 through the resolved root only, and `EndProcess(12)` is called.
   - `emptyRoot` removes `hub-one` through the link, inside the TempDir.
   - Without the fix, pid 12 does not match and `endedProcess` is nil.

   Correct.
10. `TestSandboxVerbs` clear subtest. The orphan has `Dir=root`; `/proc` reports the resolved cwd, which the resolved-root branch matches. `emptyRoot` keeps the root directory itself, so the orphan's cwd is not pulled out from under it before the signal. Correct.

**Findings:** none.

**Tool calls:** none. Everything I traced was already in context.
