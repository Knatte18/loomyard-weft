# Add `lyx report convergence`: wakes, stops, rounds, fresh-code fixes

## What changes

A new read-only verb, `lyx report convergence`, reports per finished run and per ISO week (UTC):

- what woke the orch, by kind, and the steps batten took by itself;
- stop and failure classes, and how many runs each recurred in;
- arbiter repairs and the steps an LLM took by hand (goto, reroute, unexplained resume, hand landing);
- parent decisions added after Discussion;
- review rounds per segment and their outcome;
- landings on the parent branch by kind, and the share of fixes that repair freshly landed code, plus a separate `unmapped` share for direct pushes.

It answers the four questions on the board entry: do orch wakes per run fall, do the same stop and failure classes recur, does the fresh-code fix share fall, and do review rounds per segment fall.
Every count derives from records lyx already writes; no orch, driver or agent keeps new bookkeeping.
Output is a JSON envelope by default and `--format markdown` renders one section per question.
Flags are `--since <YYYY-MM-DD>` (default twelve weeks back) and `--fresh-days <N>` (default 3); a malformed date, a window below 1 or an unknown format is refused before any read.

## Design decisions

- The verb runs from the hub's prime only and refuses elsewhere.
  `internal/reportcli` resolves the geometry and tells `internal/reportengine` every path it reads, so the engine never imports `internal/lyxcwd`; `pattern/PATTERN-told-geometry.md` lists it.
- Everything is read from git objects at refs resolved once: archive tags, the prime's records branch, the board branch and the parent branch.
  The verb writes nothing and takes no lock.
- A run is one `archive/<slug>/<sha>` tag.
  Only full loom runs enter per-run figures; other tags count per week as `partial`.
- An absent record is unknown, never zero.
  Weeks state how many runs lack each source, and a figure with no data reads as no data.
  A record that exists but does not parse becomes a finding naming run and file, and the exit code stays 0.
- Tolerance of unknown fields is confined to the report: each owning package gained a tolerant parse-from-bytes function, and no strict reader that a live run or gate calls changed.
- Wakes are classified in `internal/battenshed` (`classify.go`), beside the writer of the state-change text.
  The Land ready-tasks notice leaves no git object, so that kind is always absent.
- The halt-note format moved into the stdlib-only leaf `internal/haltnote`, which `internal/loomcli` writes through and the report parses through.
  `PATTERN-sole-parsers.md` names it the sole parser and `PATTERN-leaf-packages.md` lists it.
- Landings map to runs by an exact join on the `Warp-SHA` trailer (fast-forward, merge or squash), then by a time join bounded by the Finalize checkpoint commit and the board's done commit.
  A fast-forward landing's own task-branch commits are neither landings nor `unmapped`.
- Fix kind comes from the board labels last recorded before the landing: `bug` is `fix`, `feature` or `enhancement` is `feature`, else `other`.
- A fix is `fresh` when more than half of its blamed pre-image lines (Go test files, `docs/` and markdown excluded) landed within `--fresh-days` before it.
- Drive reports are counted per run, not parsed, which departs from the board entry's "from the drive reports" since their bodies are LLM prose.
- `internal/gitrepo` gained tag listing, a first-parent range walk, subdirectory listing, pre-image lines and first-parent blame.
- `docs/overview.md`, the module `doc.go` files, the sandbox core suite and `cmd/lyx/testdata/test-budget.yaml` are updated for the new packages.

## To check by hand

- Run `lyx report convergence` and `lyx report convergence --format markdown` on a real hub with history, and check the figures against a few runs by eye.
  Only the `hubforge` hub test exercised the real wiring.
- Seed parsing: the decision record says `run-robustness` lands first and this change then drops `shedrun.ParseSeedTolerant` in favour of the tolerant `ParseSeed`.
  `ParseSeedTolerant` is still present in `internal/shedrun/seed.go`; confirm the order of landing and remove it if `ParseSeed` is already tolerant on `main`.
  Also confirm `internal/alignment/records.go` still compiles after that merge.
- `review-archive-siblings-counted-as-segments` (`internal/reportengine/rounds.go`): check that timestamped review-archive sibling directories are not counted as segments.
- `weekly-aggregates-omit-repairs-and-llm-steps` (`internal/reportengine/weeks.go`): check that the weekly aggregates carry arbiter repairs and LLM steps.
- Known limits, accepted in the design:
  - a halt resumed between two batten reads leaves no state-change entry and counts only through its halt note;
  - a clean Tier 2 run reports stop classes as absent, so weekly stop-class recurrence looks denser than it is;
  - a hand landing or direct push inside a run's checkpoint-to-done window can be taken by the time join, leaving the run's real landing `unmapped`;
  - board labels are uneven, so a mislabeled fix counts as a feature;
  - archive tag dates can move a run across a week boundary.
