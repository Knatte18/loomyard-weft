```yaml
lens: repo-invariants
outcome: reported
```

Task: the repo-invariants lens (test seams, doc text the cards make false, card-order gate feasibility) over plan cards 1–4.

**Per-card invariant list**

Card 1 (reed-overview-order-key)
- Test seam for `render.Display.Order`: held. No `//lym:seam` test names `internal/reedengine/render#Display` or any reedengine struct, and nothing calls `reflect.TypeOf` on `Display`. `Display` stays comparable with an int field, so the existing `==` comparisons, such as `orchcli/start_test.go` `spec.Display != wantDisplay`, still compile.
- New test file tier tag: held. The card adds only rows and subtests to existing tiered tests (`TestWindowOrder` t0, `TestUpdateStrandLocked` t0, `types_test.go` t0, `TestWindowOrder_RealTmux` t2) and creates no test file.
- Doc text the card makes false: held.
  - The `reedengine/doc.go` "Overview anchor", "Window order", status-bar and Alt+Up/Down paragraphs are carried by card 1.
  - No `PATTERN.md` entry or `pattern/` file mentions the overview.
- Card order: held. It is the first card. The one test that asserted the removed refusal (`strand_test.go:120` `SecondOverviewUnderAnotherNameRefused`) is removed on this card. `orchcli` and `loomcli` reach reed through fakes or unchanged specs, so their tests still pass in the state after card 1. That includes the `loomcli` smoke subtest "start beside a runs strand adds the status strand in its own window", because the add spec is unchanged until card 4.

Card 2 (reed-overview-focus-fallback)
- Test seam: held (no struct field added).
- New test file tier tag: held (rows and subtests only).
- Doc text: held.
  - The `reedengine/doc.go` FollowSession paragraph (lines 81–82) and the focus-rule sentence (line 413) are carried.
  - The `FollowRun` sentence (line 419) is carried.
  - Line 408 ("the overview, the shell and hidden windows are not agent windows") names a category, not a count, so it stays true.
- Card order: held. Its tier-2 subtest is appended after card 1's subtests, which come from an earlier card.

Card 3 (runs-strand-beside-status)
- Test seam: held. `RunsStrandOrder` is a const and adds no field.
- New test file: none.
- Doc text: held for package docs.
  - `orchcli/doc.go:13` and `orchengine/doc.go:14` are carried.
  - `beamcli/doc.go:76` only says start brings up "its runs strand" and stays true.
  - `PATTERN.md` and `pattern/` mention nothing about this.
- Card order: held. It needs only card 1's `Display.Order`. Dropping the `loomengine` import from `runs.go` matches the removed lookup, and `LoomStatusStrandName` keeps its `loomcli` users.

Card 4 (status-strand-second-overview)
- Test seam: held (const only).
- New test file: none.
- Doc text: held. `loomcli/doc.go:62` is carried and is the only `loomcli` package-doc sentence about where the status strand goes. `shedengine/doc.go:207` mentions only the loom-status spec schema and is unrelated.
- Card order: held. `orchcli.RunsStrandOrder` comes from card 3, which is earlier.

**Findings**

None. No broken invariant in this lens.

Tool calls:
1. Grepped for `//lym:seam` and `reflect` over `Display`: the context did not show whether a reflective seam test covers `render.Display`.
2. Grepped `PATTERN.md` and `pattern/`, and found the `doc.go` files that mention the overview, runs or loom-status strands: the context had no repo-wide `doc.go`, `PATTERN.md` or `pattern/` sweep.
3. Read the overview lines in `beamcli`, `shedengine` and `orchengine` `doc.go`: two of those files had not been read before.
