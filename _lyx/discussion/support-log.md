# Support log — run-pane-timing

## Interview

### Turn 1 — exploration and scope (autonomous)

Read the board entry, the related notes (`report-timing`, `viewer-off-shell`, `reed-status-buttons`, `loom-model-key-names`, `burler-fixer-fork`, `run-verbs-shared`) and the code around the checklist, statuspane, batten's overview, shuttle's gate outcome, the Bouncer and Burler roles and `agentname`.
Findings that shaped the questions:

- The checklist serves loom and darn through `loomrecipe.Rows`; batten has no per-run pane, only the prime runs overview.
- `statuspane.FormatMinutes` is chosen so the pane's text changes at most once a minute and its poll backs off; each redraw erases screen and scrollback.
- Shuttle reports only a re-prompt count per gate entry; the findings file and run directory are deleted on finalize, so gate time is not in any record today.
- The Bouncer roles are constants; Burler's role prefixes are constants; the spawn spec's segment is the color segment, which is `review` for three of the four loom perches.

### Turn 2 — questions and auto-picks

No operator answers in autonomous mode; each question below was decided by auto-pick, recorded in the ledger.

1. Seconds on a live row? Recommended: closed spans only. Picked.
2. Round total: sum of the round's lines. Picked.
3. Numbering: entry index in recipe order, perch one number. Picked.
4. Batten's pane: the runs overview stage cell. Picked.
5. Gate time source: new shuttle record carried into history, versus shipping item 3 partially. Recommended: new record. Picked.
6. Segment in role names: explicit recipe key versus derived from the shed segment label versus the color segment. Recommended: explicit key. Picked.

### Turn 3 — lens review

Six lens forks returned reports; all findings judged, duplicates merged.

- premises/scenarios/pattern-fit — Webster's verify gate never reaches history; only `MultiLLMProducer` and `BurlerFixProducer` carry gate counts: fixed, D5 names the carriers and makes the Webster row one.
- premises/goal-scope/scenarios — persisting spans across re-attach is unsupported, unasked and races Attach's own `run.json` write: fixed, spans stay in `Wait`'s memory like the attempt count; loss on re-attach is an Open risk.
- premises/scenarios — the old-role probe risk is false, Attach matches on output files: fixed, risk replaced by "keeps its unprefixed name"; D8 states the attach fact.
- premises/pattern-fit/scenarios — fold and reroute entries ignore the new field: fixed, D5 keeps today's fold test, appends spans on fold, covers reroute entries; acceptance added.
- premises — Burler roles come from the seat table's role prefix: fixed, D8 names the profile path.
- premises/goal-scope/pattern-fit — `loomrecipe.Rows` refuses other recipes, so "no checklist change" overstated and batten has no rows: fixed, D3 names one lookup over every embedded recipe, in Scope.
- scenarios — a rework span left open by deadline, session loss or terminal failure: fixed, D5 ends it at finalize; acceptance added.
- scenarios — gate spans ignore pauses: fixed, D5 records spans with start and end and D6 takes out the pause coverage; acceptance added.
- scenarios — D7's "next visit" can match the wrong visit, and "review segment" was ambiguous for PR-Review: fixed, D7 takes only the entry right after the bounce and defines the segment as one holding a Bouncer row.
- scenarios — `role_prefix` unbounded beyond grammar: fixed, D8 checks it with `agentname.ValidateRole` and leaves collisions to `NumberRole`.
- scenarios — copy on every outcome, empty list, unknown name, negative spans: fixed in D5, D6 and acceptance.
- goal-scope — D1's pattern rationale is wrong, the pattern does not forbid a file-reading pane's redraw: fixed, rationale is now the scrollback wipe, and D1 states it narrows item 1 for the running row.
- goal-scope — roles outside review segments left out with an unchecked claim (arbiter, conflict, treadle): fixed, Out states the reason (one row or stop at a time, the checklist names the row); not extended, since the board's complaint is the identical review-segment names.
- goal-scope — "in the pane" redefined against the board's usage: fixed, the assumption now states the checklist already names segment rows.
- goal-scope — child number in batten's stage cell ambiguous: fixed, D4 uses the dotted `4.7` form.
- goal-scope — persistence is scope creep: fixed with the persistence finding above.
- pattern-fit — role names in stencils, docs and a claims test: fixed, D8 puts them in scope.
- pattern-fit — `role_prefix` must be accepted by each row's decoder, not only `shedbuild`: fixed in D8; the PATTERN-sole-parsers line removed.
- pattern-fit — child seed read must go through `shedrun`: fixed, added to Constraints.
- board-fit — `tools/tokencount` tallies by role: fixed, D8 states tallies split per segment; Open risk added.
- board-fit — pointers to `report-timing`, `loom-model-key-names`, `run-verbs-shared`: fixed in Notes.
- board-fit — `burler-fixer-fork` may remove the fixer strand: rejected, that note is unbuilt and D8 binds only the sessions that exist.
- board-fit — `orch-context-overview` and `llm-agnostic-engine` overlap: rejected, no conflict, and the run-record addition is gone.
- leanness — D8 and D1 rationales argue rejected alternatives: fixed, comparisons dropped.
- leanness — auto-mode bullets repeat decisions: fixed, the two duplicate bullets cut.
- leanness — Constraints restate inlined PATTERN entries: fixed, trimmed to entries a decision leans on.
- leanness — docs bullet lists files: fixed, one line naming the rule.

## Rejected alternatives

- Seconds on the open visit too: wipes the operator's scrollback every second for the whole run.
- Shipping item 3 only as far as today's records allow: the board permits it, but the gap is one record shuttle alone can fill.
- Deriving gate time from file mtimes or `wait.yaml`: the run directory is deleted on finalize and the wait marker is display-only and removed on entry.
- Persisting gate spans in `run.json` across re-attach: Attach already rewrites that record, and the attempt count resets on re-attach anyway.
- Bare durations instead of spans: the checklist could not take paused time out of them.
- Deriving the role prefix from the shed segment label (`discussion-review-burler-review`): long, and stripping `-Review` would parse a label that has no contract.
- Using the color segment as the prefix: three loom perches share `review`, so it does not distinguish them.
- Prefixing arbiter, conflict and treadle roles: each serves one row or stop at a time and the checklist names it.
- Numbering every recipe row, Bouncer and Burler rows separately: contradicts the board's "a perch takes one number".
- Showing the child's row number undotted in batten's stage cell: reads like a batten row number.
- Adding a per-run checklist pane for batten: no such pane exists and the overview already is its pane.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Seconds on live rows — auto-pick: closed spans with seconds, open time in minutes (D1), a narrowing of item 1. Rationale: scrollback wipe on every redraw.
- Q2 Round total — auto-pick: sum of the round's visit lines (D2).
- Q3 Numbering rule — auto-pick: recipe-order entry index, review segment one entry, off-recipe rows unnumbered (D3).
- Q4 Batten numbering — auto-pick: stage cell in the runs overview, dotted for the child (D4).
- Q5 Gate timing — auto-pick: shuttle records spans per entry in memory, carried to the history entry (D5, D6); row-gate rework read from history routes (D7).
- Q6 Role naming — auto-pick: optional `role_prefix` recipe key on Bouncer and Burler rows (D8).
- Q7 What "the pane" means for item 5 — auto-pick: the session names reed shows; the checklist already names segment rows.
- Q8 Roles outside review segments — auto-pick: unchanged, reason in Scope Out.
