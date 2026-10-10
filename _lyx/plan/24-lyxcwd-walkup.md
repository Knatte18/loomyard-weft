# Card 24 — lyxcwd-walkup

**Edit:**
- `internal/lyxcwd#gitWorktreeRoot`
- `internal/lyxcwd#Resolve`
- `internal/lyxcwd/leaf_enforcement_test.go`
- `internal/lyxcwd/lyxcwd_test.go`
- `internal/fabricengine/warplayout_test.go`

**Uses:**
- `plan:internal/dotgit#FindRoot`
- `plan:internal/dotgit#ErrNotRepository`
- `internal/hubforge#SharedHub`
- `internal/hubforge#Shape`

**Prosa:**
- `docs/shared-libs/lyxcwd.md`
- `pattern/PATTERN-cwd-resolution.md`
- `internal/gitexec/doc.go`

**Intent:** D12: `lyxcwd` finds the worktree root in-process and spawns no git on the happy path.
`gitWorktreeRoot` calls `dotgit.FindRoot(cwd)` and returns the cleaned root; any error maps to the bare `ErrNotAGitRepo` sentinel, so every consumer that pins its exact rendering keeps matching, and a pruned worktree or a gitfile naming a missing git dir is refused with the same message, as `rev-parse --show-toplevel` exits 128 on them.
The function keeps its name and signature, so `resolveCore` and `resolveWithAnchorCore` are untouched; `Resolve`'s doc comment stops naming the spawn.
`lyxcwd` no longer imports `gitexec`, so the leaf allowlist becomes `internal/dotgit` alone, and the package comment and `docs/shared-libs/lyxcwd.md`'s dependency and `Resolve` paragraphs say so; `PATTERN-cwd-resolution.md`'s last line becomes "imports the standard library and `internal/dotgit` only", and `gitexec`'s doc drops the sentence that `logger` reaches it through `lyxcwd`.
The recorded-anchor read and the anchor gate are unchanged.
Tests in `lyxcwd_test.go`, integration-tagged because the fixtures are real repositories: `TestResolve_AnchorScenario` gains steps for a linked worktree added with `git worktree add`, which resolves to the linked root with the same `Location` fields as today; a gitfile rewritten to a relative `gitdir:`, which resolves the same; a subdirectory holding an empty `.git` directory, which resolves to the enclosing root and still fails the gate with `ErrCwdOutsideAnchor`; a gitfile whose target was removed, and a linked worktree whose `commondir` target was removed, both refused with `ErrNotAGitRepo`; `TestResolve_NotAGitRepo` is unchanged and pins today's message.
Spawn-freedom is pinned structurally by the leaf test: with `gitexec` out of the import set the package cannot spawn.
With the spawn gone, `fabricengine`'s `TestWarpLayoutFor_FastPathMatchesResolveWorktree` makes only reads: it compares `WarpLayoutForForTest` with `lyxcwd.ResolveWorktree` on the prime, both in-process.
So it takes `SharedHub` of the `.` no-pair shape under the overview's hub-fixture classification, and its file header stops calling the `ResolveWorktree` fallback spawning.

**ImpactSummary:** One function body changes behind an unchanged signature; every `Resolve` caller in the tree sees the same `Location` and the same errors, and `lyxcwd`'s import set shrinks by one; one fabricengine test moves its fixture onto the shared hub.

**Verify:** lyx gate test --tags integration ./internal/lyxcwd ./internal/fabricengine -- -run 'TestResolve|TestWarpLayoutFor'
