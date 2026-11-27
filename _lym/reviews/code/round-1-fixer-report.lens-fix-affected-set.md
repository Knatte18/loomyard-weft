```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set check of the fixer's change set, re-running the searches behind the set-premised findings.

Left unchecked: none. F1, F2, F4 and F5 were re-searched. F7 and F8 were checked member by member from context, and every named branch has a row or assertion. F3, F6, F9, F10 and F11 are single-site premises.

Findings:

- **LOW**: `internal/gateslot/template.yaml:18`, the sibling of the F4 change. The template still says `upkeep (scope marked, run by the board-tidy agent on the main branch)` with no condition, yet `hubDobbyDepsStarting` (`internal/beamcli/dobbydeps.go`) now renders no upkeep call when `cfg.Scoper != "go"`. This comment ships to every hub, and the template's own default scoper is `none`, so under the default the stated behaviour does not happen. `dobbyengine/doc.go` and the no-upkeep fragment were updated, but this line was not.

Set results with no finding:

- **F5**: the unmarked `cmd/lym` tests that read files or spawn are fixture-tree or own-package tests. I saw no other unmarked whole-repository scan.
- **F2**: no added line in the diff names a config key in short form as a key. The remaining "the go scoper" wordings are prose.

Tool calls:

- Grep of the added diff lines for short config key names: my context held no diff-wide search for F2.
- `awk` over `cmd/lym` for unmarked tests that read files or spawn: my context had only the `scankit` file list, not a per-test marker check.
- Grep for texts describing upkeep and the marked scope (output truncated to a preview): checks F1 and F4 siblings not yet read.
- Narrower grep over `gateslot`, `impactset`, `contracts`, `pattern` and `docs` for upkeep texts (also previewed): same purpose.
- Grep of the unmarked spawn-reference, driver-stencil and CLI-tree tests for outside reads: those files were never read.
