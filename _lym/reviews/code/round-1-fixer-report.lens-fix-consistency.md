```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set against its surroundings for stale siblings and twins (fix-consistency lens).

**MEDIUM — five new `//` comment lines are 120 characters or more.**
- Changed text:
  - `internal/shuttleengine/doc.go`, the new line "The clear goes only into a box that reads blank or unreadable with no operator active, so text typed since…" (142 characters).
  - `internal/shuttleengine/doc.go`, the line "A command's text starts no turn: a turn start past the paste whose instruction is that text means…" (131).
  - `internal/shuttleengine/send.go:63`, the `sendContext` comment line "its paste follows a clear of the box, and a turn start past the paste that carries its text…" (133).
  - `internal/shuttleengine/send.go`, the `clearBoxBeforeCommand` doc line "The clear goes only into a box that reads blank or unreadable, with no operator active in the pane within the quiet window:" (123).
  - `internal/shuttleengine/session_test.go:737`, "A turn start the paste writes with the command's text as its instruction means the provider took the command as a prompt:" (121).
- Evidence: recent commit `74ea6b01` ("Comment lint: flag `//` comment lines of 120 characters or more") adds a lint for this, and the prose skill's rule is under 120. I measured the lengths on the added `//` lines of `git diff HEAD~5`. Each line needs a semantic break at a clause boundary.

**LOW — the `ErrCommandTakenAsPrompt` doc is stale against the new match rule.**
- Changed text: `send.go` around lines 592-597, "marks a pasted command … whose paste started a turn".
- Evidence: after the fix, only a turn start whose instruction equals the pasted text yields this error. The sibling comments at `send.go:63`, the `confirmPasted` doc and `doc.go` all state the instruction match. "Whose paste started a turn" now reads as any turn, which is the premise of F2.

No other stale sibling found. `grep` shows `claudeengine/doc.go:422`, `creelcli/delivery.go:460` and the new `creelengine/doc.go` and `creelcli/doc.go` sentences agree with the new behaviour. `clearBoxBeforeCommand` has one caller, `pasteVerified`, already updated to pass `reader`.

Calls made:
1. `grep` for `clearBoxBeforeCommand`, "taken as/for a prompt" and "as a prompt" across `*.go` and `*.md`. My context lacked the sibling mentions outside the files the fixer read.
2. `git diff HEAD~5` filtered to added `//` lines of 120 characters or more. My context lacked their measured lengths.
