```yaml
lens: fix-proposal
outcome: reported
```

Task: test the reviewer's proposals the fixer carried out by building a concrete case where each one goes wrong.

1. **MEDIUM. D1 (F12) and D2 (F13): a caller that loses the lock to a resuming worker waits forever.**
   - **Changed text:** D1 says "The caller follows another pid's record only when it is unfinished and its start is later than the caller's own start." D2 says "A resuming worker rewrites only the pid, start ticks and state … keeping the done steps and stored inputs."
   - **Case:** an interrupted record stands. Two `lym adopt X` calls run at the same time, and each spawns a resuming worker. The winner keeps the record's original "its start", because D2 lists that field apart from the start ticks, and a resume does not rewrite it. That start is earlier than the losing caller's own start. The losing caller therefore never follows the record and "keeps waiting under the same wake rules". In a plain shell, D1 says the follow waits until the worker ends, so this caller hangs while the winner runs.
   - **Fix:** a resume should also rewrite the record's start, or the rule should compare the record's start ticks.

2. **MEDIUM. D10 (F1) with D1 (F10): `lym s` never starts the beam of a subject that adopted fully, when another subject of the same record was held back.**
   - **Changed text:** D10 says "When an unfinished record lists the subject in its resolved subject set, `--here` takes the record's request as its own." D1 says "non-zero when it ends `stopped`".
   - **Case:** `lym adopt A B` holds B back on a local-commit stop. A reaches `check`, and the record ends `stopped` (D3). In A's worktree, `lym s` reads the record, which lists A, and resumes it. The resume is past `owner`, so it starts at `done` and ends `stopped` again. The caller exits non-zero, and the shortcut stops before `beam start` (D10). A is owned here and fully adopted, yet `lym s` there can never start it until B is settled or discarded.
   - **Compare D4:** D4's cover leaves out subjects "among the subjects whose owner write stood". D10's test has no such exclusion.

3. **LOW. D10 (F1) with D4 (F2): `lym s` in a beam that inherits root's owner during `lym adopt root` refuses instead of following.**
   - **Changed text:** D10 says "lists the subject in its resolved subject set". D4 says "every beam worktree whose owner it reads through `hubgeom.ParentOf` from the subject".
   - **Case:** `lym adopt root` is running. In a beam with no owner of its own, the record's resolved set does not list the beam, so `--here` pulls Bolt and reads the inherited foreign owner. It then runs `lym adopt <beam>`, and D2 refuses that as "other subjects". `lym s` in that beam fails during, and after an abandoned, root adopt. D4 covers that beam but D10's record test does not.

Further tool calls: none.
