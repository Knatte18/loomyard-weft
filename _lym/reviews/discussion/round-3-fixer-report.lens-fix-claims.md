```yaml
lens: fix-claims
outcome: reported
```

Task: check that every bound, behaviour or code fact the round-3 fix added or rewrote holds against the code at HEAD.

**[LOW] D6 pane-ancestry bullet, "also refuses, as creel's resolution does, a caller in no known pane while any live strand's pane process is unknown"**
Summary: the sentence describes lym's check as working like creel's resolution, but creel has a second "cannot tell" case that the sentence leaves out. When the hub's strands cannot be read at all, the outcome of lym's check is unstated, so it could still let a caller through.
Evidence: in `internal/creelengine/identity.go:120-121`, `paneOfPeer` returns `unreadable` when `strandsErr != nil`, and `resolveSender` (`:68-69`) refuses on any non-empty `unreadable`, not only on an unknown pane process (`:129-131`). The fix's D2, D6 and acceptance wording names only "a live strand's pane process is unknown".

The other added claims hold:
- **D2 rationale, `hubWriteRefusal` and machine-layer writes:** `hubwide.go:49-61` lets a slug-free name through from the prime worktree. `doc.go:16` and `doc.go:25-28` confirm that a machine-layer write passes the same mistaken-write guard.
- **D2 setup file, "through `internal/state`, with owner-only permissions":** `state.WriteJSON` sets no mode, but it ends in `fsx.AtomicWriteBytes`. That function writes through `os.CreateTemp` (mode 0600) and then renames (`fsx.go:60-77`), so the file ends up owner-only. This works only as a side effect of `CreateTemp`, not through any setting in `internal/state`.
- **D3 Bound, a hand-run `lym slack relay` from a non-orch pane is refused:** `identity.go:63-64` refuses a program claim from a non-orch pane.
- **D9, a hidden verb carries the `internal` audience:** `cmd/lym/clitree_test.go:216-217` requires a hidden command to have audience `internal`.

Calls made:
- grep of `internal/state` and `internal/configcli/doc.go`: to check the setup file's permission claim and the machine-layer guard claim, neither of which was in my context.
- Read of `internal/state/state.go:25-104`: to find what file mode `WriteJSON` uses.
- grep of `internal/fsx` for `AtomicWriteBytes`: to find where the mode is set.
- Read of `internal/fsx/fsx.go:48-87`: to confirm the temp-file-and-rename mode.
