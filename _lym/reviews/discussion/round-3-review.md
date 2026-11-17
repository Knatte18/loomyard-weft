---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:89-96
    summary: D4 states HTML escaping for entry bodies only, so how the page renders the other entry fields from window.BOARD is left unbound
    fix: In D4, after the bullet on the Markdown subset, add one bullet saying the page inserts every other field it renders from window.BOARD (handle, slug, title, brief, labels, project titles and briefs, the facts line, and the design and decision file names used in a link) as text and never as markup, and that a file name enters an href only as a relative path with its double quotes escaped. Add to the D4 Review acceptance criterion that a title or brief holding an HTML tag renders as literal text.
---

### [MEDIUM] D4 escapes bodies only; other entry text from the data file is unbound

**Location:** `_lym/discussion/decision-record.md:89-96` (D4), `_lym/discussion/decision-record.md:235-238` (D4 criterion)

**Issue:**
D4 says "Every body is HTML-escaped before the subset applies", and the D4 criterion lists "escapes HTML in bodies".
The page also renders title, brief, handle, slug, labels, the facts line and project titles and briefs from `window.BOARD`.
Those fields are free text an entry's author, including an agent, controls.
D1 closes the `</script>` hole in the data file, but nothing in D4 says how the page puts those fields into the DOM.
A viewer that sets row markup from a title or brief with `innerHTML` runs a tag or an event handler from the board's text, in a page opened from `file://`.
The rubric's attack-surface question asks a Decision that adds a new surface to state what bounds it.
D4 bounds the body's link and markup path and leaves every other rendered field unstated, and the in-browser behaviour is a review check only, so no test catches it.

## Rejected

- Focus 1, D6's workspace seam: not a defect.
  `internal/fabricengine/hubworkspace.go` already owns `beamWorkspaceMark` and already hands the file's shape to `vscode.BuildWorkspace`.
  D6 has fabric read only the hook members' names to refuse that mark, and the `tasks` key and the `folders` and `settings` refusal sit in `internal/vscode`.
  That is the placement `PATTERN-fabric-boundary` asks of a change that touches fabric's existing `internal/vscode` import, and the arbiter's added entry already settles it.
- Focus 2, silent reveal: the record no longer claims it.
  D6 sets `"reveal": "always"` with `"focus": false`, the Rejected alternatives name the silent-reveal option as dropped for lacking a source, and D5 needs no exit-status statement.
- Focus 3, added entries against the main text: no Decision, criterion, auto-mode assumption, Scope bullet, Goal clause or Open risks line builds or carries the dropped README folds.
  D7 survives only in the added entries' own text, and the Notes for the plan writer names it as dropped.
- Focus 4, D5's `--board-path` and cwd account: correct against `internal/boardcli/cli.go`.
  The pre-run takes the flag branch with `location` nil and no layout, and otherwise resolves through `lymcwd.Resolve`.
  `lymcwd.ResolveWithin` accepts a cwd inside the anchored directory or the records sibling and refuses one outside it with `ErrCwdOutsideAnchor`, and `lymcwd.Resolve` stays exact for `rerender`.
- Focus 5, D1's escaping sentence: the raw bytes of the sentence hold the words "a backslash and `u` followed by the hex code `003c`, `003e` or `0026`" and no backslash character.
  Go's default encoder writes those three escapes, and the D1 criterion, which looks for no literal `</script>` or `<b>`, agrees with it.
- D6 premises checked in code and found true: `lym beam start root` names the prime, `ideengine.LymBinary` is a package variable defaulting to `selfexec.LymBinary`, `InstallPairFolderNamer` is installed in `cmd/lym/main.go`, and the keybinding table in `internal/vscode/keybindings.go` holds `ctrl+alt+a` for `reed attach`.
- D3's replay premise checked in code: `FabricReplay.Claims` claims by `isBoardOverview` and the design and record paths, so the two new names are unclaimed today and need the claim D3 adds.
