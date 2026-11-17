```yaml
lens: fix-consistency
outcome: reported
```

Findings (lens fix-consistency):

1. **LOW** — `internal/boardcli/doc.go:68`, a twin of the changed text, was left as it was.
   - It still reads "placed before the first project or note".
   - The fixer changed the `list` help text (`cli.go:933-934`) and the `withRootProjectRow` doc to "before the first project, note or done entry, and last under --sort".
   - The package doc also says an entry lacking the date comes last, so it now disagrees with the code on done entries and on `--sort`.
   - Evidence: grep shows `doc.go:68` as the only other place with the old placement wording.

2. **LOW** — `internal/boardcli/verbs_test.go:346` has a stale comment.
   - It reads "stands before the first project or note", which now omits the done-entry rule the test pins later.
   - It is the only stale comment of this kind in the test file.

Not findings:
- `plugins/ly/skills/board/SKILL.md:28` states no placement, so it is unaffected.
- `docs/research/.../report.md` is a dated research report, so it is out of scope.

Calls:
1. `grep` for the old placement wording across the repo's `.go` and `.md` files, because my context lacked `doc.go`, the skill file and the docs.
2. `sed -n 62,72p internal/boardcli/doc.go` and a `grep` for P0 in the `.md` files, to confirm the package doc wording and rule out other docs.
