```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the F1 and F2 sets, meaning every checklist item against wave 1 and every file-read phrase for the at-definitions limit, and check the fix covers each member.

I checked the F1 set by walking each checklist bullet (lines 11-17) against the edited wave 1 sentence. These are covered: target files, callers, callees, other-process paths, writers and readers of states, the board entry, `pattern/` files, and failure, restart and resume paths.

**LOW. `burler-fork-reading.md` wave 1 (line 22) still names no call for the output bullet (line 14).**
- Line 14 asks for "every test, caller, doc and fixture that reads or asserts" each changed output, key, string, file shape or behaviour.
- Line 21 lists outputs among the things to enumerate. Wave 1 issues "one search per symbol for its callers, tests, docs and fixtures" and "one search per state or setting for its writers and readers", and neither names an output.
- A literal reader gets no first-wave call for output readers and assertions. This is the same kind of gap as F1's premise, and the fix left it open.
- Suggested fix: change the per-symbol search to "one search per symbol or output".
- The review did not name this member. Its premise was "a checklist item no wave names", and this item is not named.

The F2 set holds: the changed phrase is the only whole-file-style read instruction in the waves. Wave 2 reads "every definition and call site", and line 28 states the limit.

I made no tool calls, because my context already shows the changed file.
