```yaml
lens: fix-consistency
outcome: reported
```

Checked the change set for agreement with its surroundings (lens: fix-consistency).

no findings

What agrees with the new text:
- **Code:** `content` puts `title` only in the `Name=` key and, through `cmdFile`, the `rem` line. `Comment=` comes from `verb`. This matches `doc.go:19-21` and `desktoplauncher.go:138`. `cmdFile`'s own doc comment ("titled `lym: <title>`") does not claim the title is displayed.
- **File names:** "A `.cmd` launcher shows its file name" agrees with the naming paragraph at `doc.go:17-18` and with `hubPath` and `allHubsPath`.
- **Tests:** the rows of `TestAdd_WritesLaunchersRunningMachineStart` expect `Name=lym: ...` on freedesktop and `rem lym: ...` on both `.cmd` forms.
- **Other docs:** the launcher passages in `docs/overview.md` (the tree line and the paragraph near line 912) and `internal/dobbyengine/doc.go` say nothing about what a launcher displays, so no matching text was left unchanged.
- **Landing summary:** the reworded lines 6-7 agree with `doc.go`. The later "both displayed names" and "new hub name on each form" sentences still hold.

Tool calls:
1. `git grep` for other statements about launcher names and display: my context had only the package files and the summary.
2. `sed -n` on `docs/overview.md` and `internal/dobbyengine/doc.go`: the grep cut or only located those passages.
