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
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
---

## What changed

- F1: `startOpener` on both platforms logs the opener's exit with its args and the wait error, then delivers it.
- F2: the spawned server's stdout and stderr both go to the server log.
  `spawnServer` now resolves the binary and calls `spawnServerBinary`, so a test can pass a stub;
  `spawnserver_linux_integration_test.go` asserts both streams reach the log.
  A stub stands in for a failing `serve`: failing a real one through `open` waits out the whole start bound.
- F3: a successful action moves a dirty form to the answered revision only when the form had the revision the action sent.
- F4: rows in `TestServer` for the own `Origin`, a stale promote, a listed decision file and project design file, and an empty `after` on a board with no store.
- F5: the cap selection is `openerCapFor`, tested by `TestOpenerCapFor`;
  `TestBoardServeScenario` asserts the server's run-process record under the hub owner.
- F6: an action button does nothing while its entry has an action in flight.
- F7: reworded both comments in `internal/runproc/end.go`, and the same claim in `internal/runproc/doc.go`.
- F8: a `TestRegenerateHubWorkspace` row.
  The hook makes the beam's anchor unreadable during the hub file's call, since an anchor unreadable from the start drops the pair from the listing.
- F9: a `TestRenderBoardData` case with its own task list.
  The shared fixture is left alone: the README writes an unheld dependency in a code span, which its place comparison rejects.
- F10: `inline` refuses a link whose raw target holds a placeholder, before `safeTarget`.

## Verification

`lym gate test --tier 0 --tier 1` over `boardcli`, `boardserve`, `boardengine`, `runproc` and `fabricengine`: no failures, and the new tests ran in their tiers.
`go vet` passed on those packages, and `GOOS=windows go vet ./internal/boardcli` passed.
The page script changes (F3, F6, F10) were not executed: no test runs the page's JavaScript.

## Disputed

None.

## Deferred

None.

## Rulings

None.
