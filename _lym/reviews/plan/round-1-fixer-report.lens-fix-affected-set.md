```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each set-premise finding's search and check the fix covered every member.

Unchecked findings: F3, F5, F6 and F10 have no set premise. F2 was checked against the cards in context, with no new search.

- LOW. Changed text: card 1's new `render.go` header bullet and card 3's matching bullet (F7). F7's premise is the set of comments that list the board's render outputs. The fix covers `render.go:2` and `replay.go:31` but not `internal/boardengine/doc.go:339`, the `# README` section: "Every write renders README.md, the overview of every project, and one board file per project." After cards 1 and 3 a write also renders `board-data.js` and `board.html`, so that sentence goes stale. Card 1 adds a separate paragraph about `board-data.js` and holds `doc.go` under Extends, not Edit. Card 3 says only that the section "names `board.html` as a fixed render output". Neither card says the line 339 sentence changes. Evidence: `internal/boardengine/doc.go:333` (`// # README`) and `:339`.
- F1: "every op but `create`" covers edit, promote, release, draft and undraft. These are all of card 4's non-create ops, with no gap.
- F8: no other doc lists `internal/` packages one per line. The grep found only plan-spec examples and benchmark tables, so `docs/overview.md` is the only member.
- F9: the only other closed list of hub roles is `subjectPassSpared` (`internal/runproc/end.go:100`). Card 6 already keeps `RoleBoardServer` out of it. `record.go` documents each constant on its own, with no closed list. Covered.

Tool calls:
1. Grepped `boardengine` and `boardcli` comments for render-output and replay file lists (F7). My context held only `render.go:1-10` and `replay.go:20-44`.
2. Grepped `.md` files for `internal/boardengine/` and `internal/boardcli/` (F8). My context held only `docs/overview.md:285-305`.
3. Grepped for `RoleSlackRelay` (F9). My context held only `runproc/doc.go`.
4. Read `doc.go`'s section headings and lines 336-342 to confirm line 339 sits in `# README`, since call 1 gave only the line.
