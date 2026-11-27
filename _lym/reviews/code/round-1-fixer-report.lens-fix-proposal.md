```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1 and F2, both applied as proposed.

no findings

I checked the proposed wording against `content` and `cmdFile` and found no case where it's wrong:
- **Linux:** `Name=lym: ` + title is written for both the hub launcher (title = shortname) and the all-hubs one (title = "start every hub").
- **Both `.cmd` forms:** `FormWindowsWSL` and `FormWindows` both go through `cmdFile`, which writes `rem lym: <title>`. The file name comes from `hubPath` or `allHubsPath` and never from the title.
- **Neighbouring text:** the new `doc.go` lines, the `content` doc comment, the `cmdFile` doc comment and the edited sentence in `_lym/landing/summary.md` agree with each other.

Tool calls: none.
