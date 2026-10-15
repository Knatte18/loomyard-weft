# Card 1 — statuspane-format-seconds

**Create:**
- `plan:internal/statuspane#FormatSeconds` -> `func FormatSeconds(d time.Duration) string`

**Edit:**
- `internal/statuspane#TestFormatMinutes`
- `internal/statuspane/doc.go`
- `docs/overview.md`

**Intent:**
Add `FormatSeconds` to `internal/statuspane/pane.go`, beside `FormatMinutes`, the seconds-precision format D1 names.
It renders a duration below one minute as `45s`, below one hour as `12m05s`, below one day as `1h05m12s`, and from one day on exactly as `FormatMinutes` does (`2d03h`), by delegating to it.
A negative duration renders as `0s`, the way `FormatMinutes` floors at `0m`.

`TestFormatMinutes` becomes the one table test of both formats: each row names the formatter it calls, so the package's test count stays at its budget row.
Its new rows pin `FormatSeconds` at `0s` for a negative duration, `45s`, `1m00s`, `12m05s`, `59m59s`, `1h00m00s`, `1h05m12s`, `23h59m59s`, `1d00h` and `2d03h`, and the existing rows keep pinning `FormatMinutes`.
Its doc comment names both formats.

The package doc's last paragraph says `FormatMinutes` is the one time format both panes use.
Rewrite it: `FormatMinutes` is the format of every time that is still growing, so a pane's text changes at most once a minute and its poll backs off between changes, and `FormatSeconds` is the format of a closed span, which never changes and so costs no redraw.
`docs/overview.md`'s module-table row for `internal/statuspane` calls the time format minute-resolution; make it name both formats.

**ImpactSummary:** one new exported function and one test table in `internal/statuspane`; no caller changes yet, the checklist adopts the format in a later card.

**Commit:** 1: statuspane: FormatSeconds for closed spans
