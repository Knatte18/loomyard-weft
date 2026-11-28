Lens fix-proposal: I challenged the proposals the fixer carried out as written (F1, F3, F5, F6, F8, F9, F12, F13, F14) and the parent rulings (F2, F4, F11). Two findings.

**1. MEDIUM — card 4, the `RemoteOnly` bullet added for F13.**
- Changed text: "The engine asks it first with `hasOrigin` alone and nil tips, and a false answer ends with no further read … Only after that does the engine read the branch, the head and origin's tips and ask it again."
- Summary: under the bullet's own rule, a first call with origin present always answers false, so the remote-only read never runs.
- Evidence: the same bullet says "no remote branch of the checkout's name, read empty". `RemoteOnly(true, nil, branch)` finds no entry for `branch` in nil tips, so it must answer `read` false. The new sentence makes a false first answer end the flow, so the branch, head and tips reads never run when origin exists.
- The proposal works only if `RemoteOnly` treats nil tips (not read yet) differently from an empty map (no such branch). The card does not say so, and it keeps the signature.
- Fix: say that nil tips with `hasOrigin` true answers `read` true. Or split the call into a `hasOrigin` check followed by the tips verdict.

**2. LOW — card 4, the codec-planting text added for F9.**
- Changed text: "it writes a status holding two history rows through `state.UpdateJSON` over `shedengine.Status` at `shedrun.StatusFile` in the pair's records worktree".
- Summary: the planted file may not sit at the tree path that `OriginCodeTip` reads.
- Evidence: `shedrun.StatusFile` joins `RunDir(l, runID)`, which uses `runSegment(l, runID)` (`internal/shedrun/paths.go:147-149,200-201`). The reader reads `TreeRunFileRels(runID)`'s status half, which is `<lymdir>/shed/<runID>/status.json` with the bare runID (`paths.go:554-560`). Today's `plantRunTransition` commits at that `TreeRunFileRels` path, not at `StatusFile`.
- Unverified: I did not find `runSegment`'s body (grep in `paths.go` returned nothing), so I could not confirm whether the segment differs from the slug for a pair. Planting at `TreeRunFileRels(slug)`'s status path inside the records worktree would match the reader whatever the segment is.

No case found against F1, F3, F5, F6, F8, F10, F12, F14, or the F2, F4 and F11 rulings. For F3, `mapPlainPushError` handles `ErrPushRetriesExhausted` first (`internal/gitrepo/push.go:276`), ahead of the `ErrPushRejected` wrap at line 292. So mapping an exhausted push to its own class ahead of transport, and recording it as `transport`, matches today.

Tool calls:
1. `gitrepo/push.go:85-100`, `pushretry.go:125-140`, and `shedrun`'s `StatusFile`/`TreeRunFileRels`: how an exhausted push error is wrapped (F3), and the status paths (F9). Neither was in context.
2. `shedrun` `runSegment`/`RunDir` and `gitrepo` `mapPlainPushError`: the path segment (F9) and the order of the push-error mapping (F3). Not in context.
3. A grep for `func runSegment` in `shedrun/paths.go`: its body, for F9. It returned nothing.
