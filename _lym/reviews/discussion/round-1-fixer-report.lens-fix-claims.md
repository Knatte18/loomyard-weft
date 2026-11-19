```yaml
lens: fix-claims
outcome: reported
```

I checked the fix's new claims about the code against HEAD; four findings.

1. **MEDIUM.** The claim in D5's second bullet does not hold. It reads "A server removes any leftover state file right after it takes the lock and writes its own only once its port is bound (D11), so the state that answers is the live server's."
   - D11's exit bullet has the old server release its lock, wake the long polls, and only then stop taking requests.
   - So a new server can take the lock before the old server stops accepting, and before the new one has removed the old server's state file, which the exit leaves on purpose.
   - An `open` in that window gets an answer from the old, draining server, which accepts its own token. It then hands over an address that is about to close.
   - The guarantee needs the exit to stop taking new requests before, or at the same moment as, it releases the lock.

2. **LOW.** D5's bound claims xdg-open "runs the first browser of `$BROWSER` or its built-in list without backgrounding it". No source read this round backs that.
   - The fetch of the xdg-utils script returned an access-denied page, and the record names no version.
   - The script's generic mode also tries `xdg-mime`/desktop-file, `run-mailcap` and `mimeopen` before the `$BROWSER` list, so "the first browser" misstates the order for a local file.

3. **LOW.** D6's new pin says it was "read on VS Code's `main` branch on 2026-11-19". But the folder-order loop and the missing-label picker came from a model summary of the unread middle of `abstractTaskService.ts`, not from verbatim source. The pin overstates what was read.

4. **LOW.** D11 claims "A hub takeover by another machine ends it … through `runproc`, since its role is outside `subjectPassSpared`". This holds only if the takeover path calls `EndSubject` with the hub owner.
   - The callers I found are `pairteardown.EndSubjectProcesses` (`internal/pairteardown/endsubject.go:40`), passed `subjectOwner(geom)` in `internal/beamcli/dobbydeps.go:303`, and `EndSparing` in `internal/fabriccli/anchordeps.go:258`.
   - None of them visibly passes `runproc.HubOwner`, so nothing I read shows that a hub takeover reaches a `HubOwner` record.

Claims that hold: `boardCriticalSection` exists (`internal/boardengine/board.go:83`). `boardengine.IsRunStatus` and the `" · "` separator exist (`internal/boardengine/runlock.go`). `fabricengine.PrimeName` exists (`internal/fabricengine/worktreelist.go:178`). `shuttleengines.KeepWarm(dirs, strand)` exists (`internal/shuttleengines/keepwarm.go:31`). The DACL and doc.go citations are correct. `lymcwd.Resolve` requires the cwd to equal the anchored directory (`internal/lymcwd/anchor.go:121`), and `vscode.WriteConfig` puts `.vscode` under the anchor, so the task's cwd passes.

Tool calls:
- Grep in `internal/boardengine`, `internal/fabricengine` and `internal/shuttleengines`, to confirm the symbol names the fix cites. My context did not hold them.
- `lymcwd` `buildLocation` gate lookup, to check that `Resolve` accepts the task's cwd. My context had only `resolveCore`.
- `checkCwdAnchorGate` and the `WriteConfig` body, to read the gate's exact rule and where `.vscode` sits. Neither was in my context.
- Grep for `EndSubject` and `EndSparing` callers, to check the takeover claim. My context had only `runproc`'s spared list.
