```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1 to F8), looking for a case where the proposal is wrong or breaks something elsewhere.

**MEDIUM — `contracts/specs/loom-plan-spec.md:128` (F8 as proposed)**
- **Summary:** The spec now says a `Create` target under "the worktree anchor's git-ignored `.scratch/`" is skipped by `create-not-done`, but the code skips only a root-relative `.scratch/` prefix.
- **Evidence:** `internal/planglyph/donecheck.go:29` sets `scratchDirPrefix = ".scratch/"`. `donecheck.go:104` applies it with `strings.HasPrefix(key, scratchDirPrefix)`. `DoneChecks` receives only `worktreeRoot` and has no anchor.
- **Concrete case:** at anchor `backend`, a card targets `backend/.scratch/draft.md`. That path does not start with `.scratch/`, so it gets a done-check, contradicting the new spec sentence.
- **Sibling text still disagrees:** `internal/planglyph/donecheck.go:27`, `donecheck.go:62` and `internal/planglyph/doc.go:137-138` still say "worktree root's `.scratch/`". `doc.go:138` states outright that "a `.scratch/` below the root ... keep[s] [its] done-check". The review's fix also told the fixer to "check Webster's `create-not-done` skip for a path rule that assumes the root", and that rule does assume the root.
- **Proposal fault:** the one-phrase swap makes the spec state a behavior the code does not hold. At anchor `.` it happens to be true, since the anchor is the root. At a subpath anchor it is false.
- **Resolution:** either change the skip to cover the anchor, or word the spec so it stays true, for example by naming the root-relative `.scratch/` path.

The other proposals I challenged held up:
- **F1 (Lstat census):** the case I built, a sibling with a real or foreign root `.scratch`, now keeps the line. Any error other than `fs.ErrNotExist` also answers true. The new test row covers the retained-line case.
- **F4 (new anchor-change row):** the prime is re-anchored to `sub` and the mover goes from `.` to `other`, so no sibling holds the root. That is the only layout where the drop is observable, and the added `/.scratch` assertion in the first subtest covers the sibling-retained branch.
- **F5, F6, F7, F3, F2:** I found no breakage elsewhere. The helper changes (`trackScratchFile` now uses a relative path, `writeLegacyScratchExclude` now uses an anchor-based line) keep the anchor `.` behavior identical.

Tool calls: none. My context already held every changed file and the review.
