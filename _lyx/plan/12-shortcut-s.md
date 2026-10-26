# Card 12 — shortcut-s

**Edit:**
- `internal/shortcutcli/shortcuts.yaml`
- `internal/shortcutcli#testShortcuts`
- `internal/shortcutcli#fakeRootFactory`
- `internal/shortcutcli#TestShortcut_Run`
- `internal/shortcutcli#TestMount_File`
- `cmd/lyx#TestShortcuts_FileAndRootListing`
- `internal/shortcutcli/doc.go`
- `docs/overview.md`
- `docs/loomyard-overview.svg`

**Intent:**
D4: `shortcuts.yaml` gains `s: [[beam, start], [reed, attach]]` and drops `prime`, which card 10 already repointed at `beam start`, so `s` replaces it.
`lyx s` takes no slug: every word goes to the last argv, `reed attach`, as the routing does today, so it acts on the acting worktree's beam; a refusal of `lyx beam start` exits non-zero, attaches nothing and leaves its envelope on the terminal, and on success the attach covers the output, both by the existing stop-on-failure dispatch.

`testShortcuts` mirrors the shipped `s` and `fakeRootFactory` fakes `beam start` where it faked `orch start`; `TestShortcut_Run`'s rows name `s` and assert `beam start` then `reed attach` on success, and that a failing `beam start` stops the sequence with its exit code and runs no attach.
`TestMount_File` asserts the shipped file mounts exactly `a`, `s`, `p` and `r`, and `s`'s help reads `shortcut: lyx beam start, then lyx reed attach`; `TestShortcuts_FileAndRootListing` asserts `s` in the root's Shortcuts listing and in the help index.
`shortcutcli/doc.go`, `docs/overview.md`'s shortcuts line and the overview diagram name `lyx s`.

**ImpactSummary:** one embedded file entry swapped; the shortcut routing code is unchanged and every test pinning the shipped set is on this card.

**Commit:** 12: Add the `lyx s` shortcut and drop `lyx prime`
