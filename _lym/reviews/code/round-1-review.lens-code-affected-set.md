```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: I checked what the diff's changes force outside the lines it touches.

Per change, the set I found:

1. **Four outcomes in `contracts/orchrole/role-intake.md`, with the new close-as-already-fixed rule (lines 26–38).**
   - The fork's return-line rule at the end of the same file allows `closed` with a close reason. That covers the new outcome. Covered.
   - The pointer in `contracts/orchrole/orch-role.md:33` ("`role-intake.md`: when the operator asks for intake") names no outcome count. Covered.
   - The rendering wiring is `contracts/orchrole/orchrole.go:22,37`. It embeds the file and the file's shape is unchanged. Covered.
   - The "No denied recovery" rule in `pattern/PATTERN-refusal-way-forward.md` holds. The new step reads with git and never checks out or changes anything. None of the forms on the closed deny list are named. The shellguard publish row (`internal/shellguard/rows.go:404`) refuses only git push and gh writes, so `git log` and `git show` are allowed. Covered.
   - `pattern/PATTERN-stencil-ownership.md` holds. The orch-role exception keeps `role-*.md` embedded and never copied to the board, and the diff edits it in place. Covered.
   - `pattern/PATTERN-code-conventions.md` (no hard-coded branch) governs code paths. The contract prose says "the hub's main branch", not a literal `main`. Covered.
   - Wherever else "three outcomes" or "close as noise" appears, it is about something unrelated. Nothing else describes intake's outcomes. Covered.

2. **The `close` command's Short, Long and Examples in `internal/boardcli/intake.go:117,128–132`.**
   - The test `internal/boardcli/payload_test.go:41` checks the payload against its declared keys, not the help prose, and the payload is unchanged. Covered.
   - `TestHelpSchema_LeafCommands` (from the reach report) reads the help schema. The `Use` string and payload are unchanged. Covered.
   - The `plugins/ly/skills/board/SKILL.md` example block at lines 230–231 has the same new example, word for word. Covered.
   - The new example's reason is free text containing "on main". It is a sample string in help, not a code path choosing a branch, so `PATTERN-code-conventions` does not apply. Covered.

3. **The `plugins/ly/skills/board/SKILL.md` example (line 231) and the close bullet (lines 246–247).**
   - The bullet still opens "`close` ends a noise issue…", and the next line adds the already-fixed case. Together they are consistent. Covered.

**Declared PATTERN changes:** the directive declares none, so there is nothing to check.

**Commit messages and departures:** the only commit, 997f952cd, has no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailers. Its message matches the diff. Covered.

**Stencil wording:** the diff keeps every existing rule in `role-intake.md`. It changes "three" to "four" and adds one bullet, and drops or weakens nothing. Covered.

No findings.

Tool calls:
1. A grep of the repo for other text describing intake outcomes, the close as noise, or `intake close`. My context held only the three diffed files and `payload_test.go:41`.
2. A read of `internal/shellguard/rows.go:385–410`, plus a grep for "default branch" wording, to confirm git reads are not denied to the orch fork. My context had only the line-404 grep hit.
