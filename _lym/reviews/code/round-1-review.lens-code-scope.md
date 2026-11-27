```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each hunk to the board entry's goal points and checked each point for a change that builds it.

**Decisions** (decision record alone; the record has no `Added after Discussion` entries):
- **G1**: make the in-place edit match also read the commands after a heredoc's closing delimiter, while still skipping the heredoc body.
- **G2**: add a test row for `sed -i` and `perl -pi` on a line after a closed heredoc.

**Per change:**
- **C1** `internal/shellguard/rows.go:315-349`: adds `heredocBodyLinePattern`, `closedHeredocPattern` and `afterClosedHeredocScope`, splits `inPlaceEditCommandPattern` into `inPlaceEditCommandIn`, and adds the after-heredoc alternative. Traces to G1.
- **C2** `internal/shellguard/rows.go:835-838`: the shell-edit row comment now states the new reach and the false denials. Forced by C1, since the old "So does anything after a heredoc operator's line" would be stale.
- **C3** `internal/shellguard/doc.go:22-26`: the package doc states the new reach. Forced by C1.
- **C4** `internal/shellguard/rows_integration_test.go:194-201`: eight new rows.
  - Four deny rows trace to G2: `sed_in_place_after_a_closed_heredoc` and `perl_in_place_cluster_after_a_closed_heredoc` (the entry's own shape).
  - The `<<-` and separator deny rows also trace to G1's reach.
  - The four allow rows trace to G1's "still skipping the heredoc body".
- **C5** `internal/shuttleengine/claudeengine/testdata/settings-golden.json`: the rendered pattern. Forced by C1, since `TestBuildSettings_MatchesGolden` pins the settings byte for byte (`settings_test.go:1150-1200`).
- **C6** commit `bdc61d8cd`: re-breaks lines in the C2 and C3 comments. Forced by C2 and C3.

No change is unrequested scope.

**Per decision:**
- **G1**: built by C1, with C2, C3 and C5 forced.
  - The reach clause is built for the top level.
  - The clause "while still skipping the heredoc body" is built only for bodies with no line that holds a bare word alone (see finding 1).
- **G2**: built by C4, which has rows for both `sed -i` and `perl -pi`.

**Findings:**

1. **MEDIUM, scope.** `internal/shellguard/rows.go:324` (`closedHeredocPattern`). G1's clause "while still skipping the heredoc body" is only partly built. A body line that holds a bare word alone ends the skipped body, so any in-place edit spelling on a later body line is read in command position and denied.
   - Evidence: I ran the rendered HEAD hook from `settings-golden.json` (`.scratch/rv/cases2.txt`).
     - `cat > .scratch/msg.txt <<'EOF'\nSummary\n\nsed -i is now refused after a heredoc\nEOF\ngit commit -F .scratch/msg.txt` is denied.
     - `cat > .scratch/notes.txt <<EOF\nFindings\nperl -pi slipped past the guard\nEOF` is denied.
     - The base hook allows both.
   - The row comment (`rows.go:837-838`) and `_lym/landing/summary.md` "Design decisions" call this a design choice. The record has no `Added after Discussion` entry licensing it, so it is a deviation from the entry, not a recorded call.
   - Fix: either bring the body skip in line with the entry, or have the parent record the trade-off with `lym loom decision add`.
2. **LOW, scope.** `internal/shellguard/rows.go:329-333` (`afterClosedHeredocScope`). G1's "read the commands that follow a heredoc's closing delimiter" is not built for a `sh -c` script after the closing line. The entry's own shape is covered.
   - Evidence (`.scratch/rv/cases.txt`):
     - `cat <<EOF\ny z\nEOF\nbash -c 'sed -i s/a/b/ f'` is allowed, as is the double-quoted form. The same `bash -c 'sed -i s/a/b/ f'` with no heredoc before it is denied.
     - A heredoc delimiter holding `-` or `.` (`<<'END-X'`) and two heredocs on one line (`cat <<A <<B`) are also never read past.
   - The scope derives from `shellScopes[0]` alone and has no `shellScriptReachPattern` step.

Tool calls: none. Everything this lens needed was already in my context.
