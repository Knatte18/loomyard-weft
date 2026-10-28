# Support log — VS Code colours by project and worktree, and lym ide seed with no slug

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- Recording the beam's colour in the beam pair's `.vscode/settings.json` alone: the pair is made later by the beam's run, and the file is machine-local, so another machine would pick a different colour.
- Recording it in the hub scratch directory: machine-local for the same reason.
- Picking the beam colour in `beamcli` from a board read before the write: two concurrent creates could take the same colour; picking inside the owner write's lock cannot.
- Keeping `settings.json` write-once: the prime and every seeded worktree would keep the old green and title until the operator deletes the file.
- Finding sibling tasks through `hubgeom.ParentOf` per worktree: an origin-record read per pair, where the sibling's title-bar colour on disk already names its project.
- Writing status-bar or `window.activeBorder` keys: the board entry rules them out, since they lose colour or are not drawn in an unfocused window.

## Review rounds

_No rounds yet._

## Question ledger

- Where does a beam's colour live? Self-pick: a `beam_color` field on the beam project, written by the owner setter (D1).
- What happens to beams made before the change? Self-pick: re-running `lym beam create` records one, and the IDE picks one unrecorded meanwhile (D2, D3).
- Does seeding overwrite an existing settings.json? Self-pick: only lym's own keys (D4).
- What threshold decides black text? Self-pick: relative luminance 0.179 (D5).
- What title does a hub with no shortname get? Self-pick: the slug alone (D6).
- Does `lym ide open` lose its required slug too? Self-pick: no, the entry names only seed.
