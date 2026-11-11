```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I looked for what calls, reads or asserts it.

**Per-change list**

1. **Green mark cleared, failed mark shortened** (`check.go` `checkMarkOf` and `primeMark`)
   - Set found: the callers in `check.go`, `adopt.go` and `doc.go`; the tests in `check_test.go`, `adopt_test.go`, `check_integration_test.go`, `checkview_test.go` and `deps_test.go`; and the docs and contracts (README, orch-role).
   - Result: covered.
   - `checkMarkOf` and `primeMark` are asserted only by `TestRunCheckMarks` and `TestPrimeMark`, and the diff updates both.
   - The other `checkGreen` uses (record state, check view, deps) don't read the mark.
   - No stencil, contract or README text describes the old green mark text.
   - `SetCheckMark` with empty text unsets all three tmux options (`internal/reedengine/checkmark.go:24-30`), so the zero mark clears the bar.

2. **Per-check envelope line in `suiteMessage`**
   - Set found: `adopt.go:258`, the `adopt.go` doc comment, the `doc.go:376` envelope description, and the `adopt_test.go` message rows.
   - Result: covered.
   - A grep for "progress and result" outside the tests finds nothing left behind.

3. **`attach` and `up` refusal** (the pre-run in `cli.go`, the `attach.go` and `up.go` help text, and `doc.go`)
   - Set found: callers that run `lym reed up` or `attach` from a non-worktree cwd, and docs that describe the verbs.
   - Result: covered.
   - Every in-repo caller runs the verbs from a task worktree or the prime.
     - `burlercli/run.go:194-198` uses `reedUp` for standalone mode, which resolves its own session; I did not trace its cwd any further.
     - `contracts/orchrole/orch-role.md:342` says `cd <worktree> && lym reed attach`.
     - `pairteardown/teardown.go:595` says "run from <anchor>".
   - The README, the contracts and the stencils name no non-worktree use.

4. **PATTERN entries**
   - `PATTERN-reed-panes` covers spawning and pane text and is not touched by a session-boot refusal.
   - `PATTERN-refusal-way-forward` covers only webster, shed and loom refusals; reed is outside its list, and the refusal names its way forward anyway.
   - `PATTERN-fabric-boundary` is respected: the pre-run calls only the exported `fabricengine.RequireDrivableWorktree`.
   - The prompt declares no PATTERN changes, and the diff has none.

5. **Commit messages and departures**
   - No commit carries a `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

**Tool calls**
1. `grep` for the old mark and envelope texts and the `checkGreen` consumers across the repo — the manifest held no repo-wide search.
2. `grep` for `lym reed attach` and `up` mentions, and `ls` plus `grep` of the reed pattern file — to find docs and callers.
3. `grep` for callers that invoke `reed up` or `attach` and for status bar mentions in the README and contracts — to find in-repo callers and docs.
4. `Read` of `internal/burlercli/run.go:180-215` — to check the standalone `reedUp` caller; it only calls `c.reedUp()`, so I did not trace the cwd it runs from.
