```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: rerun each set-premised finding's search and check the fix covers every member.

**Findings left unchecked:** F2, F4, F5, F6, F9, F11, F12, F13, F14, F15 and F16. None of them has a set of tests, callers or sites as its premise.

**Checked:**
- **F3, call sites of the new prime-name parameter.** Every caller now passes the name: card 10's `start` and `away` through the ensure seam, card 11's `relayStarter`, card 12's tick and card 13's `hubAnchorDeps`. Every relay argv in cards 9 and 10 and in the overview carries `--prime`.
- **F7, creel comments that claim a single coded refusal.** `identity.go:46` and `doc.go:225-227` are the only members, and both are covered. `client.go:132` is generic and stays true.
- **F8, doc and help texts made stale by cards 11 to 13.** The fix covers every member:
  - `orchengine/doc.go:35`, the one-line stencil
  - the bring-up steps in `orchcli/doc.go`
  - `beamcli/doc.go:209` and `:212`, the `Deps.Tick` text and its Bound
  - `fabriccli/doc.go:110` and `anchor.go:530`, `dobby_start_error`
  - the beamcli line in `docs/overview.md`
- **F10, planned tests that pin one unexported helper.** After the fix, every planned test targets a public surface or an exported symbol.
- **F1, writers of `RelayState`.** Steps 3, 4 and 5 and the starter are all covered by the sentence that every write is a locked read-modify-write.

no findings

Tool calls:
1. Grepped the plan cards for relay-start call sites and argvs that lack the prime name. The changed signatures span cards my context did not search as a set.
2. Grepped `creelengine` and `creelcli` code comments for "only refusal", "no other refusal" and "carries a code". The code comments outside the two cited lines were not in my context.
3. Grepped the beamcli, orchengine, orchcli and fabriccli docs, `anchor.go` and `docs/overview.md` for the stale texts. Those files were not in my context.
4. Grepped the plan cards for planned tests and unexported helpers. I needed the full test set across cards, not only cards 7 and 8.
