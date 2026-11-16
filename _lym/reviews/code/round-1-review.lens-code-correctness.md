```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function and branch the diff adds, I ran a concrete case and judged the result.

**Function and branch list**

- `parseWindow` (retrocli/cli.go):
  - slugs with `--since`: refused, correct.
  - `--max-runs 0`: refused, correct.
  - `--since yesterday` and `--until 2026-13-01`: refused, correct.
  - no flags: Window{Now, MaxRuns 10}, correct.
  - `--until` set to a date after today: accepted with no check. Wrong downstream (F1).
- `fileRetro`:
  - repo with no `tools/retro`: refused, correct. A stat error other than not-exist is reported as "holds no" (minor, not filed).
  - ReadTasks fails: error returned, correct.
  - report dir missing: no names, correct.
  - plan slug already on the board: refused, but the way forward is wrong for named or capped plans (F3).
  - UpsertTask fails: error, and nothing was released, correct.
  - SetStatus fails: the draft stays and the message names the release step. Correct.
- `resolvePrime`: non-prime worktree refused, `PrimeName` error wrapped. Correct.
- `reportDirNames`: absent dir gives nil, read error is wrapped, file entries are included and harmless. Correct.
- `landedRuns`: an unclaimed landing (Slug "") is dropped. Since is the window start, and `assemble` drops commits before it. Correct.
- `openBoard`: config error returned, strand name from env. Correct.
- `LastRetro`:
  - nil input: (zero, false), correct.
  - `retro-2026-11-16` fails the layout and is skipped, correct.
  - `retro-notes` skipped, correct.
  - several stamps: the newest wins, correct.
- `Window.Start`: slugs give Now-60d, `--since` gives Since, an earlier retro gives last, otherwise end-7d. Correct.
- `Select`, bare window:
  - from=last=09:30: runs 10:00, 11:00 and 12:00 are kept. 12:00 is relanded and keyed by its newest landing. Correct.
  - zero runs: refused, correct.
  - cap 2: the oldest two are kept and until moves to the last kept landing. A tie at the cut is wrong (F2).
  - `--until` in the future: wrong (F1).
- `Select`, named:
  - an unknown slug is refused, duplicates are deduped, a count over the cap is refused. Correct.
  - From is the oldest named run's own landing, and the stencil says "after" From (F4).
- `sortByLanding`, `newPlan`, `ReportDir`, `Slugs`, `TaskFields`: correct.
- `Body`: fills every marker the stencil uses, and the tokencount `-out` given twice resolves to the last value. Correct.
- `retro.sh`:
  - HUB and RECORDS both set: prime is never read, and `${HUB:-…}` does not expand it, so `set -u` is safe. Correct.
  - run with no archive tag: written to `missing.jsonl`. Correct.
  - run with a tag but no status.json: no rows and no missing entry. Silent but harmless.
  - facts files: the newest file per perch is picked by number. Correct.
  - transcripts are read after usage, so the usage join for that slug works. Correct.
  - the live run over two real runs exited 0.
- `facts.awk`, against a real round-2 facts file (12 columns): the header is mapped and each data row's cells are emitted. The recurring-keys line "No ledger key recurs" is ignored, and the "Keys open in an earlier round" section is excluded. Correct.
- `transcript.jq`:
  - turn grouping by message id, a text-only turn (zero calls, so no lookup), and refusal detection all correct.
  - gate fail on `Exit code 1`: correct.
  - a `done:false` result falls back to the background notification, defaulting to not failed. Correct.
- `figures.jq`:
  - medians: an even count averages the two middle values, an empty list gives 0.
  - `money` precedence is right.
  - outliers are listed only for rows with at least three sessions.
  - verify kinds "gate" and "rework" match `GateSpanKindGate` and `GateSpanKindRework`.
  - all correct.
- `reportcli` rename: the call sites are identical and behavior is unchanged. Correct.

**Findings**

- F1 — MEDIUM, design. `internal/retroengine/retro.go:760-776` (`Select`) and `internal/retrocli/cli.go:342-356` (`parseWindow`).
  - Summary: a `--until` later than now stamps the retro with that future time, so every run landing between filing and that time is never covered by a later bare retro.
  - How it happens: `w.end()` returns `w.Until` unchanged (retro.go:717-723). `newPlan` stamps the slug with it (retro.go:790-791). The next `LastRetro` returns that future stamp, so `Start` begins after it (retro.go:710-711).
  - Example: with Now=2026-11-16T14:00 and `--until 2026-11-20`, the slug is `retro-20261120-000000`. Runs landing on 11-17 and 11-18 are neither in this retro, which read the landings at 14:00, nor in the next bare one.
  - Evidence: `parseWindow` has no check that until is at most now (cli.go:342-356).
  - Fix: clamp the window's end to Now, or refuse an until later than now.
- F2 — LOW, design. `internal/retroengine/retro.go:771-775` (`Select` cap branch).
  - Summary: when the cap cuts between runs that landed in the same second, the left-out run is skipped forever.
  - How it happens: `until` becomes `runs[MaxRuns-1].Landed`. The next retro's `from` is that stamp and keeps only `at.After(from)` (retro.go:763), so a dropped run with an equal `Landed` never qualifies.
  - `sortByLanding` breaking ties by slug (retro.go:780-786) shows equal times are expected.
  - Fix: on a tie at the cut, keep or drop every run of that second together.
- F3 — LOW, design. `internal/retrocli/cli.go:391-393`.
  - Summary: the duplicate-slug refusal says "file the retro again a second later", but for named-run and capped plans the slug comes from a landing time, not from now. A retry a second later builds the same slug and is refused again.
  - Example: name the run a capped retro ended on. `newPlan` stamps with `runs[len-1].Landed`, which equals the earlier retro's stamp (retro.go:757, 774), so every retry collides.
  - Fix: name a way forward that changes the slug, such as a different run set or `--since`/`--until`, or derive the message from how the slug was made.
- F4 — LOW, design. `internal/retroengine/retro.go:757` with `contracts/stencils/retro/retro-template-task.md:60`.
  - Summary: for named runs, `From` is the oldest named run's own landing, and the rendered body says "The runs landed after {{.from}}". That excludes the oldest listed run, so the body misstates its own run list.
  - Evidence: `newPlan(runs[0].Landed, …)` at retro.go:757, and the stencil line "The runs landed after {{.from}} and up to {{.until}}" at line 60.
  - Fix: make From the window start, or make the stencil sentence hold for both kinds of plan.

**Tool calls**

- `grep internal/shedengine/status.go` for the GateSpan kind values: my context did not show the kinds `retro.sh` filters on.
- `git show` of a real `round-2-facts.md` at an archive tag: my context lacked the real facts layout that `facts.awk` parses.
