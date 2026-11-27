---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The message and fresh blocks move to the rebuilt panel for a clean panel too, not only a dirty one. The proposed from-scratch rebuild of a clean panel would drop the F15 warning, since a promote changes the entry's kind and the page rebuilds the panel on the next render (internal/boardengine/board.html, panelFor)."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed-differently
    reason: "An edit is refused by the stored entry's kind (Board.GetTask), not by membership in view.Projects, which lists open projects only (internal/boardengine/view.go, BoardView.Projects), so a done project would stay editable. The EditOfADoneProject row pins it. Create is refused as proposed."
  - id: F9
    disposition: fixed-differently
    reason: "Only cmd.Stdout = logFile was added. The serve verb already prints a Run error on stdout (internal/boardcli/cli.go, outputError; cmd/lym/main.go:61 passes os.Stdout), so a second write to os.Stderr would log each failure twice."
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
---

# Fixer report

## Commits

- `ced3696de` — F1, F2, F4, F5, F6, F7, F8, F12, F13, F14, F15, F16, F18, F19.
- `bdb277140` — F9, F11, F17.
- `62fea9687` — F3.
- `bcefcdf98` — F10.
- `a76f294e3` — F20.

## What changed

- F1: `ViewEntry` has a `depends_on` field with every stored dependency, and the edit form fills from it.
- F2: the watch loop never wakes within a second of the previous wake and watches only `board.json`, named by the new `boardengine.BoardStoreFile`.
  The package doc and `PATTERN-daemon-wakeups` state the floor.
- F3: both openers log the exit.
- F4: an action on a dirty form advances its revision only when the action was sent against that revision.
- F5: an unsent edit moves to the form of the entry's new kind.
  A task that became a note has no dependency field, so typed `depends_on` text is not carried.
- F4 and F5 together: `send` takes the entry's current panel after the write answers, in case a poll rebuilt it meanwhile.
- F6, F7, F18: the fresh block, the form baseline and the retry bound, as proposed.
- F8: the server refuses a create of any kind but task or note, and an edit of a project.
- F9: the spawned server's stdout goes to the log.
- F11: `openerCap` holds the strand-to-cap choice and is tested, and the other two rows are added as proposed.
- F12, F13, F14, F19, F20: tests added.
- F15: a promote's record warning is answered as `warning` and shown in the panel.
- F16: a `Serve` failure is logged.
- F17: `open` starts a server once when the lock holder exits during the wait.

## Verification

- `lym gate test --tier 0 --tier 1` over `boardengine`, `boardserve`, `boardcli`, `fabricengine` and `cmd/lym` passed.
- After that run, one assertion in `internal/boardserve/server_test.go` was tightened, and `lym gate test ./internal/boardserve` passed.
- `GOOS=windows go vet ./internal/boardcli` passed.
- The changes to `board.html`'s script (F1, F4, F5, F6, F7, F15, F18) were never run or parsed: the repository tests do not execute it and no JavaScript runtime is installed here.
  They were checked by reading only.

## Disputed

## Deferred

## Rulings
