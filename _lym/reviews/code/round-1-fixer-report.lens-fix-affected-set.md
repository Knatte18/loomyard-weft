```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set — checked whether F1's file move and F8's docs cover every site.

No findings that rise to BLOCKING or MEDIUM. One LOW:

- **LOW** — `internal/shuttlecli/hookedit_test.go:24,30,53-55,63,77-85` is a leftover in the F1 set. The verb now takes `<edit-records-file>`, but this test still calls it an events file: the `"unused-events-file"` argument, the `eventsPath` variable, the `"events.jsonl"` file name, and the messages "seed the events file", "read the events file" and "events file = …". The test passes, but the name now misleads. It sits beside `hookedit.go`, which the fix did rename.

Everything else in the sets is covered:
- `EditRecordsFileName` is used by the writer (`settings.go`), the reader (`editloop.go`), the doc comments and the tests.
- No other edit-related "events file" wording is left in non-test code or in `docs/` outside the research archive.
- F8: the overview has both the notice (line 664) and the hook sentence (line 707); `edit_loop_window_s` is in the notice sentence.

Calls made:
1. Grep for "events file" wording near edit/hook-edit across the files and dirs the F1 fix touches: to find missed sites, which my context lacked outside the files I had read.
2. Grep for `edit_loop` / `hook-edit` / `EditLoop` in `docs`, `README.md`, `sessionsignal.go`, `sessionstate.go`, `config.go` and `template.yaml`: to check the F8 set and the config and template wording.
