# Board viewer rendered beside the README, opened by lym board open and a VS Code key

`board-viewer-build` · Task · feature, board, ide, fabric · draft

Decision record: [Sketch](decision-board-viewer-build.md)

Filed from the Sketch `board-viewer`.

Takes: the Sketch's decision record; boardengine's Render file map, RenderToDisk and its render manifest, boardView's sections, layerSection and childProjects, SetOwners' renderOwnerWrite, and FabricReplay.Claims; boardcli's PersistentPreRunE and rerender verb, lymcwd.ResolveWithin, and internal/vscode's launch_linux.go and launch_windows.go spawn as the opener pattern; vscode.WriteConfig with its reed attach task, the keybindings block and BuildWorkspace; fabricengine.RegenerateHubWorkspace and InstallPairFolderNamer as the install pattern; ideengine.LymBinary
Delivers: board-data.js and board.html as render outputs of every board write, listed in the render manifest, committed by the sync and the owner-set write, and claimed by FabricReplay; the self-contained viewer page with D4's filters, search, README grouping, expandable entries, escaping and link rules and colour scheme; lym board open with its ResolveWithin cwd, its refusal naming lym board rerender and its per-platform logged and waited opener; the board open task in every worktree's tasks.json, one beam start task in each workspace file through fabric's workspace-content hook, and alt+shift+0 and alt+shift+9 in the keybindings block
Exports: boardengine's board.html and board-data.js file-name constants and the embedded viewer; boardcli's open verb and its opener seam; fabricengine's workspace-content hook type and its process-wide Install function; internal/vscode's board open and beam start task labels and the tasks member BuildWorkspace splices after settings; the ide module's hook, installed in cmd/lym/main.go beside the pair folder namer

The part changes these packages:

- internal/boardengine (about 1200 changed lines)
- internal/boardcli (about 250 changed lines)
- internal/vscode (about 145 changed lines)
- internal/fabricengine (about 125 changed lines)
- internal/ideengine (about 100 changed lines)
- cmd/lym (about 3 changed lines)
