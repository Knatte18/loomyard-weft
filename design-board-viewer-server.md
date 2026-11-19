# Board viewer page with live editing through a loopback board server, opened by lym board open and a VS Code key

`board-viewer-server` · Task · feature, board, ide, fabric, process, cwd · running · Parts-Plan-Write

Decision record: [Sketch](decision-board-viewer-server.md)

Filed from the Sketch `board-viewer-editor`.

Takes: the Sketch's decision record whole, its Added after Discussion entries settling its Open risks bullet; boardengine's Render, RenderToDisk and the render manifest (render.go), boardView and layerSection as the README model, FabricReplay.Claims (replay.go), boardCriticalSection and its load migrations (board.go, draft.go, sketch.go), the owner-set write (ownerset.go), IsRunStatus and the run-agent rule (runlock.go, taskguard.go); creel's daemon start and lock (creelcli/spawn.go, creelengine/server.go) and dobbyengine.Live as the single-instance pattern; runproc roles and subjectPassSpared; lymdirs; fswatch.Watch; selfexec.New and LymBinary; shuttleengines.KeepWarm; githubclient/cache_windows.go's owner-only DACL as the shape to copy; vscode.WriteConfig, keybindings.go and BuildWorkspace; fabricengine.RegenerateHubWorkspace and InstallPairFolderNamer; ideengine's LymBinary seam; cmd/lym/daemonwakeups_test.go
Delivers: board-data.js and board.html as committed, claimed and replayed render outputs; the embedded viewer page, read-only at file:// and editable over http; a per-entry revision checked in the board's write section with a stale-entry refusal carrying the fresh entry; lym board serve, a per-hub loopback server with token, Host, Origin and CSP checks, the board, file and write routes, a long-polled live view, and idle and binary-replacement exit; lym board open, which starts or reuses the server and opens the page through a launcher file; the board open and beam start VS Code tasks and the alt+shift+9 and alt+shift+0 keys
Exports: the board-data.js and board.html name constants; the projection builder and the entry revision function; the stale-entry error; the expected-revision form of boardengine's writes; package internal/boardserve; lym board serve and lym board open; the board server runproc role; the board server lymdirs constant; fabricengine's workspace-content hook installer

The part changes these packages:

- internal/boardengine (about 2200 changed lines)
- internal/boardserve (about 1800 changed lines)
- internal/boardcli (about 700 changed lines)
- internal/vscode (about 300 changed lines)
- internal/fabricengine (about 160 changed lines)
- internal/ideengine (about 120 changed lines)
- internal/runproc (about 15 changed lines)
- internal/lymdirs (about 10 changed lines)
- cmd/lym (about 15 changed lines)
