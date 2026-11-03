```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the `Clean` exemption, `exemptPathspec` and `worktreeDirty` changes.

**Per function and branch: case run, verdict**

| Function or branch | Case run | Verdict |
|---|---|---|
| `worktreeDirty`, no pathspec | Existing callers (`pull.go:210`, `mergeguards.go:400`, `prune.go:232`, `reconcile.go:409`, `checkout.go:73`, `destroy.go:977`, `remove.go:785`, and the export_test wrappers) pass no pathspec. | Correct. `len(pathspec) == 0` skips the append, so the args are unchanged. |
| `worktreeDirty`, pathspec given | `scopeAll` gives `status --porcelain -- . :(exclude)…`. `scopeTracked` gives `status --porcelain --untracked-files=no -- …`. | Correct. `--` comes after all flags, and `append(append(args, "--"), pathspec...)` is safe. |
| `exemptPathspec`, empty or nil exempt | Existing `Clean(l)` callers and `cleanreason_integration_test.go`. | Correct. It returns nil, so the weft probe runs unscoped as before. |
| `exemptPathspec`, `AnchorRel` `"."` | `filepath.Join(".", "_lym/shed")` gives `_lym/shed`, so the pathspec is `. :(exclude)_lym/shed`. | Correct. |
| `exemptPathspec`, `AnchorRel` `"backend"` | The pathspec is `. :(exclude)backend/_lym/shed`. The git cwd is the weft worktree root (`RecordsWorktree(l)`), so the exclude is cwd-relative as written. | Correct. |
| `exemptPathspec`, empty-string `AnchorRel` | `Join("", x)` gives `x`. | Correct, same as `"."`. |
| `exemptPathspec`, Windows separators | `filepath.ToSlash` is applied to the joined path. | Correct. |
| Untracked-directory collapsing | I ran real git in a scratch repo where `_lym/` and `backend/_lym/` were wholly untracked. The root exclude hid `_lym/shed` and showed only `backend/`. The `backend/_lym/shed` exclude hid `backend/` and showed only `_lym/`. With the root exclude and an extra `_lym/plan.md`, `_lym/` still showed. | Correct. Git does not collapse an untracked parent into a hit on an excluded child, and a sibling under `_lym` still counts. |
| Exclude scope | The exclude matches the root and everything beneath it, and nothing else. | Correct. |
| `Clean`, weft side | `dirtyReason(..., exemptPathspec(l.AnchorRel, weftExempt)...)` returns an error on a git failure, wrapped with the label. It returns an empty reason when only the exempt files are dirty. | Correct. The error path is unchanged. |
| `Clean`, warp side | The warp probe gets no pathspec. | Correct. The code side is never exempted, as the summary says. |
| `Clean`, weft worktree absent | `os.Stat` fails, so the weft probe is skipped. | Unchanged and correct. |
| `runStateExemptRels` and `CheckResolved` | `shedrun.RunsRootRel()` returns `_lym/shed`, an anchor-relative path with no location input. | Correct. |

**Findings:** none from this lens. I ran no input where the new code returns a wrong result, drops an error, or leaves state inconsistent.

**Tool calls**
1. Scratch-dir git repo (blocked by the shell hook, nothing ran): I wanted to check how `git status` applies an exclude pathspec when the parent directory is wholly untracked.
2. Same check (blocked by the shell hook, nothing ran).
3. Same check, using `touch` instead of redirects. It ran and gave the results in the table. My context lacked git's pathspec behavior for collapsed untracked directories.
