# Card 7 — shortcuts-p-r

**Edit:**
- `internal/shortcutcli/shortcuts.yaml`
- `internal/shortcutcli/doc.go`
- `cmd/lyx#TestShortcuts_FileAndRootListing`
- `internal/shortcutcli#TestMount_File`
- `docs/overview.md`

**Intent:**
Add the short operator shortcuts `p: [tender, pause]` and `r: [tender, resume]` to `internal/shortcutcli/shortcuts.yaml`, beside `a` (Decision 12).
Bound: each shortcut runs `lyx tender pause` or `lyx tender resume` in the caller's own process environment, so tender's caller check applies unchanged (an agent may pause or resume only its own pair), and the shortcuts open no route past it.
The board entry reserves the call for the operator; these are the board's proposed names, for parent review to confirm.
`TestShortcuts_FileAndRootListing` already checks every declared shortcut against the real root, so the two names cannot shadow a module and each argv must resolve; its operator index lines gain ``- `p [<args>...]`: shortcut: lyx tender pause`` and ``- `r [<args>...]`: shortcut: lyx tender resume``, which fail if either shortcut is missing or expands to anything else.
`TestMount_File`'s shipped-file row compares the mounted shortcuts as an exact map, so it gains `"p": "shortcut: lyx tender pause"` and `"r": "shortcut: lyx tender resume"`, and its row name lists `p` and `r`.
The package doc's opening list of shortcuts, and both shortcut lists in `docs/overview.md` (the source tree's `internal/shortcutcli/` line and the shortcuts bullet), gain `lyx p` (`lyx tender pause`) and `lyx r` (`lyx tender resume`).
The tender module bullet in `docs/overview.md` changes to read that `lyx pause` and `lyx p`, and `lyx resume` and `lyx r`, are shortcuts for the first two verbs.

**ImpactSummary:** two new root commands in the shortcut group; the two tests that pin the shipped file follow it, and nothing else reads the file.

**Commit:** 7: shortcuts: lyx p and lyx r for pause and resume
