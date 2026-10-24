```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-premise finding's fix covers every member of its set.

Unchecked: none. I reran the searches for F2, F3/F10, F1 and F11. F7, F12 and F13 have no set beyond the sites the review named.

1. LOW. Concerns card 8's `--guards` bullet, now "Under `dotnet` that refusal is not built through `configRefusal`…".
   The `planTierRuns` doc comment still lists the `--guards` refusal among those "naming `lyx config gate`", and only "under the none scoper".
   Card 8 edits `internal/gatecli#planTierRuns` but never says to update that comment for the `dotnet` refusal, which no longer names `lyx config gate`.
   Evidence: `internal/gatecli/tiers.go:62`: "It refuses, naming `lyx config gate`, … and --guards under the none scoper."
2. LOW. Concerns card 10's `none` bullet, now "Under `none` the early whole return of `computeScope` becomes a per-group share…".
   The doc comment on `computeScope` still says every form is whole under `none`, and card 10 rewrites only `doc.go`'s twin sentence.
   Evidence: `internal/gatecli/scope.go:102`: "Under the none scoper each of the three forms is whole."
   Card 10's `doc.go` bullet rewrites only "`doc.go`'s sentence that every form is whole under the `none` scoper".
3. Checked, nothing found:
   - F2: every row that relies on the C# tree points at the new tree. That is card 8's computed, `--card` and `--base` rows and card 11's `Unused.cs` row. Card 12 builds its own fixture.
   - F11: the `config.go:211` refusal text is already covered by card 1's `want go, none or dotnet`.

Tool calls:
- One grep for comments naming "go or none" or the none scoper in `gatecli` and `gateslot`, plus the C# tree references in the plan cards. My context lacked the comment sites beyond `Config.Scoper` and the full list of cards that reference the tree.
