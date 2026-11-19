# Board viewer with live editing over a loopback server, opened by a key

The operator gets an interactive board page with filters, search, grouping and edit forms, opened from VS Code by `alt+shift+9`.
The page edits the board: it creates and edits tasks and notes, releases tasks, promotes notes, and drafts and undrafts tasks.
It refreshes when any other writer changes the board.
Every write goes through `internal/boardengine` like the CLI's, and carries the revision of the entry it edited.
A stale write is refused and the page shows the fresh entry beside the operator's unsent edit.

## What changes

- `internal/boardengine`
  - `Render` returns `board-data.js` (`window.BOARD = <json>;`), a view projection built by the code that builds the README, and `board.html`, an embedded viewer asset.
  - `RenderToDisk` writes both and lists them in its manifest, so every board write's sync commit carries them.
  - The owner-set write commits both.
  - `FabricReplay.Claims` claims both names, so a Bolt replay re-renders them.
  - An entry's revision is the hex SHA-256 of its JSON encoding, derived on read, so `board.json` keeps its shape.
  - The write section checks the expected revision after the load migrations and before the mutation.
    A mismatch or a removed entry returns a stale-entry error carrying the fresh projection, and saves, renders and syncs nothing.
  - The CLI verbs gain no revision flag and behave as before.
- `internal/boardserve` (new) holds the per-hub server.
  - It listens on `127.0.0.1` on an OS-picked port, and writes state (pid, port, token) with mode 0600 and, on Windows, an owner-only DACL.
  - Routes: `GET /` (the embedded page), `GET /api/board` (projection and board revision, long-polled with `after=`), `GET /file/<name>` and `POST /api/write`.
  - Checks: a `Host` header equal to `127.0.0.1:<port>`, a bearer token on `/api` and a `t` query parameter on the file route.
    A write needs `Content-Type: application/json` and an `Origin`, when present, equal to the server's own.
    The server sends no CORS header, caps request bodies, and sets `Cache-Control: no-store` and `nosniff` on every response.
    The page's CSP header allows only its own inline script and style by hash.
  - One watch loop (an `internal/fswatch` event on `board.json` or a one-second-to-eight-second timer) drives change detection, binary-replacement exit and a 30-minute idle exit.
- `internal/boardcli`
  - `lym board serve` takes the single-instance lock under a new `lymdirs` directory in the board-level `.lym`.
  - `lym board open` starts or reuses the server, writes a launcher file holding the token in the URL fragment, and hands the platform opener the launcher's path.
    The opener is `xdg-open` on Linux and `rundll32 url.dll,FileProtocolHandler` on Windows.
  - Both verbs refuse `--board-path` and refuse in a run agent's session.
- `internal/runproc` gains a role for the board server, kept out of `subjectPassSpared`.
- `internal/vscode`, `internal/ideengine`, `internal/fabricengine`, `cmd/lym`
  - `board open` goes into each worktree's `tasks.json` beside `reed attach`, and `lym ide seed` and `lym ide spawn` write `alt+shift+9` (`board open`) and `alt+shift+0` (`beam start`) into the keybindings block.
  - `beam start` goes into the workspace files' `tasks` key, so a multi-root window holds exactly one.
  - Fabric gains a general workspace-content hook, `cmd/lym/main.go` installs the ide module's implementation, and `internal/vscode` splices the returned members.
    Fabric names no workspace key but its own generated mark.

Opened by hand at `file://`, the page is read-only, with no edit control and no live refresh, and reads `board-data.js` through a script tag.
Served over `http`, it reads and writes through the API and never reads `window.BOARD`.

## Design decisions

- `lym board open` has no refusal for a missing `board.html`: it never reads the Bolt's copies, since the server serves the embedded asset.
  The `lym board rerender` line lives in the `file://` page, where a missing `board-data.js` breaks it.
- `board.html` and `board-data.js` stay committed render outputs for the read-only `file://` page, beside the server's editable page.
- Release refuses a task in a run status, naming `lym board set-status`, since a none status would make it startable while its run still holds its pair.
  Promote acts as the CLI's does.
- The server passes an empty strand name, so its writes are the operator's.
- The opener is waited on through `cmd.Wait`, not `proc.Reap`, because a failed opener must return its exit status.
  For a caller with a strand name the wait is capped at the keep-warm cap.
- The server's board config and environment are read once at start, from the prime's anchored directory whatever worktree `open` ran from.
- Any process running as the same user can read the state file and write the board through the server as the operator, a run's agent included.
  The token keeps out other users and every web page.

## To check by hand

- Open the page in a browser over `http` and at `file://`.
  The test suite has no JavaScript runtime, and `board.html` was not syntax-checked, so none of the following is covered:
  filters and OR/AND grouping, search, nesting and counts, the Markdown subset and link filtering, light and dark, the create and edit forms,
  the stale-refusal display, the retry before the `lym board open` line, and the `lym board rerender` line at `file://`.
- Press `alt+shift+9` and `alt+shift+0` in a seeded window.
  A window without the task opens VS Code's task picker filtered by the label.
- Run `lym board open` with no desktop detected: `xdg-open` can run a browser in the foreground, and the verb then waits until it exits.
- Findings a code review left open:
  - `board.html` is not pinned to LF line endings (`board-html-not-pinned-to-lf`; this diff adds a `.gitattributes` line, so confirm it covers the file).
  - The page busy-loops when `board.json` is absent (`page-busy-loops-on-absent-board-json`, in the page's request retry and `internal/boardserve`'s `GET /api/board`).
  - The verb wiring in `internal/boardcli` has behaviors no test reaches (`verb-wiring-behaviors-untested`).
  - The watch loop may lack the one-second floor `PATTERN-daemon-wakeups` requires (`watch-loop-no-one-second-floor`, in `internal/boardserve/run.go`).

## Merge danger

**Door:** one-way
**Blast radius:** board
