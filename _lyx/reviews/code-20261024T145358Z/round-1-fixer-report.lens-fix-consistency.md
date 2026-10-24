```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Checked:
- **F1:** the `lyx board profiles` entry in `internal/boardcli/doc.go:66` matches the help example at `internal/boardcli/cli.go:376` (`ok`, `defaults`, per-recipe `settings`), and the code at `cli.go:390` emits `defaults`.
- **F2:** the new `profile` help text agrees with `internal/boardengine/doc.go:59`, which says clearing a profile makes the task take its recipe's default.
- **F5:** the `decodeDefaults` comment matches the `DefaultsEntry` skip at `internal/runprofile/runprofile.go:132` and the single read at line 141.
- **F3, F4:** the new test rows agree with the template and with the existing `light: 3` row.

Outside the change set: `internal/boardcli/doc.go:54` still says `""` clears the profile and doesn't mention the default. It's the same wording the review left alone, and `boardengine/doc.go:59` covers it, so it isn't a finding.

Tool calls (1 of 6):
- One `grep` over Go and markdown files for `clears it` and `DefaultsEntry`, to find twins of the changed text that my context didn't show.
