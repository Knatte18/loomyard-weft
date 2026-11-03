---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/driftengine/spawn_test.go:110
    summary: No `failures` row pins that readChildReport takes the last error envelope or skips non-error envelopes
    fix: In the `failures` table of `TestStartCapture`, add one row whose log holds an earlier envelope marked lasting with text A, then a later `ok:false` envelope not marked lasting with text B, then an `ok:true` envelope line; want the error to contain B, not contain A, and `wantLasting` false. A version that kept the first envelope or accepted `ok:true` then fails.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftcli/cli.go:391
    summary: runCaptureChild marks every openWorktree error as lasting, including a transient read error on the recorded anchor marker
    fix: In `runCaptureChild`, set `driftengine.LastingFaultKey` only when `errors.Is(err, lymcwd.ErrNotAGitRepo) || errors.Is(err, lymcwd.ErrCwdOutsideAnchor)`, and use the plain `fail(err)` for any other `openWorktree` error, so an I/O or permission failure reading `.lym-anchor` still offers a resume. Keep the integration assertion in `anchor_integration_test.go`, which hits `ErrCwdOutsideAnchor`.
  - id: F3
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/driftengine/spawn_test.go:145
    summary: The overlong-error row uses ASCII only, so a byte-based cut would pass
    fix: In the row "an overlong error is cut to the bound", repeat a multibyte rune such as `å` instead of `x` in both the log and the want and wantNot strings, so only a rune-based cut at `childErrorMaxRunes` passes.
---

### [LOW] Last-envelope selection is not pinned

**Location:** internal/driftengine/spawn_test.go:110
**Issue:** The entry asks for the child log's last error line. Every `failures` row holds at most one envelope, so a `readChildReport` that kept the first envelope, or accepted an `ok:true` or non-string-`error` envelope, passes every row. The behaviour is the entry's core requirement and has no row that fails when it breaks.

### [LOW] Every openWorktree error is marked lasting

**Location:** internal/driftcli/cli.go:391
**Issue:** `openWorktree` returns `CwdFrom`, `Resolve` and anchor-marker read errors. The cwd refusals, `ErrNotAGitRepo` and an unresolvable cwd are fixed by the same directory on every start, so lasting is right for them. A non-not-found read error on `.lym-anchor` (`lymcwd/anchor.go:173`) is the exception: a fresh start could succeed, yet the halt says resume cannot help and drops it. The path is narrow, and the entry's "such as the cwd and anchor check" covers the common cases, so the severity stays LOW.

### [NIT] Rune-safe cut not exercised

**Location:** internal/driftengine/spawn_test.go:145
**Issue:** `boundChildError` cuts on runes, but the only cut test is ASCII, so a byte-based cut would pass.

## Rejected

- Halt names `lym drift defer followed by lym loom resume` against the entry's "drops `lym loom resume`": the defer verb's own help says to run resume after it, and the halt text states a resume alone cannot change the error. The entry is honored.
- The `lym drift capture` (FallbackDetach) message change as unlicensed scope: the entry names `startFailure` as where the message is built, and the CLI form shares it; leaving "run lym drift capture again" for a lasting error would give the same dead-end advice the entry removes.
- The new `capture-child` integration assertion as unlicensed scope: it pins the entry's second Expected bullet end to end at the real binary and adds no behaviour of its own.
- A non-string `error` envelope leaving an earlier envelope's text (code-correctness NIT): the child's `ErrFields` always writes a string, so the case is unreachable; the filter itself is covered by F1.
- Commit trailer names a different model than the harness attribution (code-affected-set LOW): the attribution is the harness's, and the diff is not at fault.
- Redundant tests: none found; the two lasting `spawn_test.go` rows reach different `startFailure` branches, and the `baseline_test.go` row reaches the `captureAgain` branch.
- doc.go edits in `driftcli` and `driftengine`: both match the landed behaviour.
