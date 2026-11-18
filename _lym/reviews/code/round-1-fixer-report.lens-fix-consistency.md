```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set for places that disagree with it, looking at sibling docs and tests.

**LOW — `_lym/landing/summary.md` "What changes", the holder bullet.**
It says "The holder leaves the shell's tree at once through a subshell". The fix for F5 also starts the holder under `setsid`, in a session of its own. The summary is not updated for that, and neither is the "Design decisions" bullet about the command keeping its process group.
The `scopeStartScript` doc comment is updated and is correct.

**LOW — `internal/proc/memorycap_linux_test.go` file header.**
The header lists what the file covers: the kill report, cleanup of what the child leaves, scope errors and cancel. It does not mention the two tests the fix adds. These check that the holder ends and that the holder is never a child of the command.
The sentence "A kill at the cap … ends what is left in the scope" no longer covers the whole file. A reader judging whether the file is the right one would miss the holder behaviour.

**LOW — `_lym/landing/summary.md` "Tests".**
It lists the tree-kill row and the scope-start test, but not the new holder tests or the unaligned-cap row. This is the summary's own record of the tests, so it is now incomplete.

Consistent with the fix:
- `doc.go:57-65` and `memorycap.go:23` say the holder keeps the scope and the command keeps its pid and process group. Both still hold, since only the holder's group changed.
- The literal `memory.max` no longer appears in non-test code outside `memorylimit.go`. The old `sleep 300 … & cat` and `read -r line <&4` text has no leftover copies in Go or markdown.
- The F3 numbers in the summary match the gate run records.

Tool calls:
1. I grepped `doc.go` and `memorycap.go` for holder, process group, setsid and `memory.max` text, because my context did not hold those files.
2. The same call grepped the repo for leftovers of the old hog command and the old holder line.
