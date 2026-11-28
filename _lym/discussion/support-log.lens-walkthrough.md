```yaml
lens: walkthrough
outcome: reported
```

## Walkthrough lens report

**Lens:** walkthrough. For each decision I ran one concrete case through its mechanism, handing each step the data the step before it produces.

### Findings

- **BLOCKING — D5 (and its D5 acceptance bullet). The needs-stored-record predicate also holds for tapestry, so every undiscussed tapestry start is refused.**
  - Evidence: D2 inlines the Jacquard block into tapestry. Under D3 the inlined rows keep their own config, so tapestry's expanded rows include jacquard's Record-Write row with `stored_only: true`.
  - D5's predicate is "its expanded rows hold a RecordWrite row with `stored_only` true". That is true for tapestry.
  - Case: `lym batten run` of an undiscussed tapestry entry reaches the start guard. The predicate is true and the entry is not discussed, so the guard refuses. The refusal's way forward then says to set recipe tapestry, which the entry already has.
  - This contradicts the acceptance bullet "false for tapestry" and D2's undiscussed-tapestry walk.
  - Fix: make the predicate "holds a stored-only RecordWrite row and no row that writes the record ahead of it", that is, no DiscussionWrite row and no motif block. Or judge it on the recipe's unexpanded top-level rows only.

- **MEDIUM — D4. The Stuck's first way forward does not recover the run that is stuck.**
  - Evidence: a jacquard run stopped at Record-Write runs under the recipe in its seed. Running `lym board upsert '{"slug":…,"recipe":"tapestry"}'` changes the board field only. It neither re-seeds nor moves the live run, so `lym loom resume` stops at the same Stuck again.
  - The way forward has to end the run and start it again under tapestry, for example a cancel with `--remove`, then the upsert, a release and `lym batten run`. That is the shape of `errorEnvelopeBodyReason` in `recordwrite.go`.
  - The `goto --to Motif` branch is right only for a tapestry run. One reason text serves both recipes, so it should say which branch applies to which recipe.

- **LOW — D2. Moving the rows changes their BlockPath, and that is not stated.**
  - Evidence: from Drift-Baseline on, every row moves under BlockPath `Jacquard`, where today it has none.
  - In-flight runs resume by row name, which is unchanged. But any reader of BlockPath, such as goto, re-launch or reports, sees new values for runs that started before the change.
  - D2's resume paragraph covers names only.

- **LOW — D7 and D6. The claim that the hub reconcile carries the new profile blocks into an existing hub is unpinned, and D6 depends on it.**
  - Evidence: if an existing hub's operator-owned `profiles.yaml` does not get the `jacquard` blocks, raddle's check reports `profile-unknown` on every jacquard part. Every split from a Sketch would then fail.
  - The record states the carry as fact without citing the runprofile reconcile rule or a test.

- **LOW — D8 and D2. The phase table timings shift.**
  - Evidence: Preflight now runs after Discussion in tapestry. In tapestry's report phase table, Preflight's time moves to after the Discussion phase, so per-phase figures for tapestry runs change across the upgrade without being noted.

### Decisions walked

| Decision | Case | Outcome |
|---|---|---|
| D1 | Jacquard run alone on a discussed entry: Preflight, then Record-Write seeds the record and the stored-record support log and commits, then Drift-Baseline onward. Plan-Write reads the seeded record. | Met |
| D2 | Discussed tapestry: the Motif gate seeds and commits, skips to the Jacquard block's SkipTo, then Preflight, then Record-Write, which writes nothing and makes a no-op commit. | Met |
| D2 | Undiscussed tapestry: Discussion runs, Discussion-Bouncer approves and stores the record (its empty on_done goes to the block exit, Jacquard), then Preflight, then Record-Write, which writes nothing. | Met, but D5 blocks the start (see finding) |
| D2 | In-flight run parked at Preflight that never entered Motif: resumes into Record-Write. With a stored record it seeds; with none it is Stuck. | Met (D4's way forward has the gap above) |
| D3 | Jacquard is the last outermost block row with empty `on_done`. Its exit is empty, so the inlined Friction-Reflect keeps on_done "" and the run ends. Plan-Bouncer, Code-Bouncer, Webster and Parent-Align all set `on_done`, so no other row silently ends the run. The Jacquard block's empty SkipTo is harmless because its gate always holds. The terminal Friction-Reflect exists after expansion. | Met |
| D4 | stored_only with the record missing and no stored record: Stuck, nothing written. With the record present: untouched. | Mechanism met; way forward does not work (see finding) |
| D5 | Undiscussed jacquard start: refused at batten's guard before any worktree is made. | Met, but the predicate is broken for tapestry |
| D6 | Proposal with a part whose recipe is jacquard: the check passes once the profile listing has a jacquard block. Filing copies the recipe, writes the record and sets `Discussed`. | Met if D7's blocks are present |
| D7 | A new hub gets the template with jacquard blocks, so the raddle profile check passes. | Met for a new hub; an existing hub is unpinned |
| D8 | Jacquard child of batten: `childRecipeSupported` admits it, and the report selects it. | Met |
| D9 | Tapestry run with `stored_record` true is keyed under jacquard. A jacquard part's raddle cost line reads the jacquard cell with stored_record true. | Met |
| D10, D11 | Text and parent action only. | Met |

### Further tool calls

1. Grepped `internal/shedengine` for entry-gate evaluation and SkipTo handling, to confirm that an empty SkipTo routes by OnDone. My context lacked the engine's skip routing.
2. Read the `on_done` lines of tapestry's Bouncer, Webster and Parent-Align rows, to check which inlined rows would end the run under D3. My context held only the row outline.
3. Read `internal/shedengine/entry.go`'s `enter`, and grepped for a recipe-graph pin on resume, to check D2's resume-by-name claim. My context lacked both.
