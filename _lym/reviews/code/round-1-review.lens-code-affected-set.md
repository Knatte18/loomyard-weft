```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines they touch.

**Per change: the set found, covered or not**

1. **`startFailure` / `childNoRecordError` shape (new `lasting` field, new error text carrying the child's error).**
   - Set: constructors and readers are `awaitRunningRecord` (`spawn.go:136,141`) and `captureAgain` (`baseline.go:144-154`). The CLI path `driftcli/cli.go:355` returns the error unchanged.
   - `loomcli/driftwiring.go:212` builds a `SpawnRequest` without `FallbackDetach`, so it gets the row-worded form with `lasting` set. That is covered.
   - Covered.

2. **Halt text (`CaptureHaltPrefix` reason).**
   - Set: the only consumers are prefix checks, in `verbs.go:190` (defer admission) and `checkmain.go:20,28` (re-capture admission).
   - The new lasting halt keeps the prefix, so `lym drift defer` and `lym drift capture` stay admitted. The grep found no stencil, spec or doc that quotes the old text.
   - Covered.

3. **`lasting_fault` key in the `capture-child` JSON envelope (new output).**
   - Set: the writer is `cli.go:391`, the reader is `readChildReport` (`spawn.go:186`), and the shared constant is `LastingFaultKey`. `output.ErrFields` leaves the key intact.
   - The integration assertion at `anchor_integration_test.go:77` and the `spawn_test` rows cover it.
   - Covered.

4. **Docs.**
   - Updated: `driftcli/doc.go:20`, `driftengine/doc.go:34` and `:84`, and the `baseline.go` and `spawn.go` doc comments.
   - One stale spot: `driftengine/doc.go:~83` still says a refusal "names `lym drift defer` with `lym loom resume`" for the precondition refusal. That refusal path is unchanged, so it is still correct.
   - Covered.

5. **PATTERN entries.**
   - `PATTERN-friction-capture` (halt writes a note) does not depend on the halt text.
   - `PATTERN-tolerant-records` applies to records, not CLI envelopes.
   - `PATTERN-named-constants` is honored by `LastingFaultKey` and `childErrorMaxRunes`.
   - `PATTERN-cwd-resolution` is untouched, since `openWorktree` still goes through lymcwd.
   - The diff declares no PATTERN changes, and the commit has no `Pattern-Departure:` trailer, so none is needed.
   - Covered.

**Findings**

- **LOW, consistency**: `internal/driftcli/cli.go:391` marks every `openWorktree` error as lasting, including `lymcwd.CwdFrom`/`Getwd` and `gitWorktreeRoot` failures. I did not check whether `gitWorktreeRoot` can fail transiently (for example, a git spawn failing under load). Where it can, the halt wrongly tells the parent that a resume cannot help. This is the same question as focus item 2; the handler should settle it. Evidence: `wiring.go:30-39`, `lymcwd.go:81,133-137`.
- **LOW, consistency**: the commit message and the `driftcli/doc.go:20` comment describe the marking as "worktree does not resolve from its cwd". That matches the code, so no mismatch, but the commit trailer (`Co-Authored-By: Claude Opus 5.5`) differs from the harness attribution (Sonnet 5.5). I'm noting it only for completeness; it is not a defect in the affected set.

No other member of the affected set is left wrong.

**Tool calls**
1. Ran `grep` across the repo, excluding `.git`, `_lym` and `.lym`, for the halt strings, `CaptureHaltPrefix` and `lasting_fault`, to find stencils, docs and consumers outside the driftengine files already in context.
