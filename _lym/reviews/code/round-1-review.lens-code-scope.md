```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens — trace each diff change to the board entry's goal points.

**Decisions (goal points of the board entry under `## Goal`; the record has no `Added after Discussion` entries):**
- G1 (#190 Expected): lym's own usage records never block worktree-clean. Either worktree-clean leaves out `_lym/usage`, or lym commits the usage records before the check.
- G2 (#194 What to do): exclude `_lym/usage/` from worktree-clean (or git-ignore it), or have lym commit or flush the usage file before Preflight evaluates the worktree.

**Changes:**
- C1: `internal/preflight/preflight.go:28-35` adds `shedrun.UsageDirRel()` to `runStateExemptRels`. Traces to G1 and G2, through the "exclude `_lym/usage`" option both issues name.
- C2: `internal/preflight/preflight.go:22-23` doc sentences about the usage directory. Forced by C1, because the function's doc lists each exempt entry.
- C3: `internal/preflight/doc.go:18-20` adds "the usage directory" to the package doc's exempt list. Forced by C1, because the package doc lists the exempt state.
- C4: `internal/preflight/preflight_integration_test.go:448-449, 476-479` writes a session usage record in `TestCheckResolved_LymOwnStateIsExempt` and updates its doc. Forced by C1 as that change's test.

**Per decision:**
- G1: built by C1, with C2, C3 and C4 forced by it.
- G2: built by C1, with C2, C3 and C4 forced by it.

**Scope check:**
- The exemption covers the whole `_lym/usage` directory, including `run.json` and `landed.json`. Both issues name the directory itself (`_lym/usage`, `_lym/usage/`), so the width is licensed and is no finding.
- No change goes outside `internal/preflight`, and no change lacks a trace.
- The other worktree-clean consumer (`preflightshed/preflight.go:47`) only reads the CheckID, so the exemption reaches it through `preflight.CheckResolved` with no extra change needed.

**Findings:** none.

**Tool calls:** none. My context already held the diff, the fasit, the commit log and the definitions.
