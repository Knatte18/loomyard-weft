```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked the record's claims and bounds. Four claims broke at MEDIUM and two at LOW.

**Findings**

1. **MEDIUM, D3 (file set).** Parent-Align's merge puts the parent's own test changes in front of the judge, and the judge will rule them `uncaused`.
   - D3 lists "files that exist at the run's baseline commit and that the committed code changes". The baseline is the branch-point capture.
   - In tapestry, Parent-Align merges the parent's upstream tip before Drift-Verify (tapestry rows 350–358). So a parent commit that widened a limit in an existing test shows up in baseline..HEAD.
   - The commit messages D3 reads are "the review range's", and the Code-Review rubric sets that range from the last merged parent tip. That range does not hold the parent commit's trailer. The file set and the commit set come from two different bases.
   - Counterexample: the parent lands a test-tolerance change carrying a `Loosened-Check:` trailer, and Parent-Align merges it. The judge sees the widened hunk and finds no trailer in the review range, rules `uncaused`, and Drift-Verify goes Stuck on work the task never did.
   - Fix: take the diff against the last merged parent tip, the same base the review range uses, or exclude merged-in parent changes.

2. **MEDIUM, D3 (judge spawn and reuse).**
   - **The spawn rule fires on nearly every run.** "The judge now spawns when there are changed existing files with no stored ruling, even with no changed cell." The list holds every changed existing file, production code included. A file with no loosened check gets no `loosened` entry, so it never has a stored ruling, and every Drift-Verify and Drift-Recheck spawns a judge.
   - **Reuse cannot work as written.** It is keyed on "the loosened hunk's text", but Go cannot know which hunk is the loosened one: only the judge picks them, and the judgement entry holds file, test, what and ruling, with no hunk text.
   - **An acceptance criterion contradicts the format.** The criterion "a judgement whose `loosened` list leaves out a listed file's ruling … is refused" assumes one ruling per listed file. The format has one entry per loosened check, and a changed non-test file has none to give.
   - Fix: a per-file judgement (none, caused or uncaused) keyed on the file's diff hash, or limit the list to files the judge marks as tests.

3. **MEDIUM, D4/D6 (re-entry target).** Rerouting to Code-Burler-Review skips the machinery that starts a new review generation and carries a review input.
   - `internal/shedadapters/amendment.go` `startNextGeneration` is the Bouncer's. It reads the review input, keeps its path only when it was written after `settled.SettledAt`, archives the settled run directory, and writes the next generation record. A full generation is chosen by `cfg.FullGeneration`, and an amendment is scoped to the change.
   - A reroute straight to Code-Burler-Review enters a run directory the Bouncer has already settled. No new generation is seeded and no review-input path is kept.
   - D6's "the re-entered Code-Burler-Review round reads the stored PR findings as a review input" therefore has no existing path to rest on.
   - Fix: reroute to Code-Bouncer as a full generation carrying the PR-findings review input, or state what new seeding the review row does.

4. **MEDIUM, D4 (fallback bound).** "Code-Review budget is 0" is not the only way Code-Review is off.
   - Sampler gates Code-Bouncer and Code-Burler-Review on `entry_gate: sampler.code_review`.
   - If a send-back reroutes into a gated-off Code-Burler-Review, the gate is checked on entry and does not hold. The row is skipped by its OnDone, and the skips chain through Code-Burler-Fix and Code-Bouncer to Drift-Recheck, Describe and PR-Review (shedengine doc, "Entry gates").
   - Result: no review runs, the findings are dropped, and PR-Review sends back again until the reroute cap halts the run after 3. The verb's fix-here test has to read whether the target row's entry gate holds, not the budget alone.

5. **LOW, D2/D3 (coverage after PR-Review).** A check loosened after Drift-Recheck is never ruled by Drift or by Code-Review.
   - Two kinds of commit land after the last Drift-Recheck: the PR-Review agent's fix-here commits and Finalize's merge-fixer commits.
   - D2 tells both roles to write the trailer, but no Drift row and no Code-Review runs after them, so the trailer is never checked.
   - The record should state this as a bound.

6. **LOW, D4 (findings file format).** Burler's verdict parser requires a verdict that agrees with the BLOCKING findings (`internal/burlerengine/verdict.go`).
   - A parent's creel findings carry no such verdict, so the agent must write one before the verb accepts the file.
   - That holds, because a refusal names the contract, but the brief should say the agent writes the verdict line.

**Claims attacked**

| Claim | Result |
|---|---|
| A reroute starts a fresh Code-Review episode (shedengine doc: it "ends the episode of every row sharing its target's Segment") | held |
| The reroute cap bounds the send-back; the count resets only on Done or a cap halt, and PR-Review never reaches Done before it reroutes | held |
| A resume after the cap grants 3 more | held, operator-gated |
| Fix-here when PR-Review declares no `on_reroute` | held |
| Fix-here when the Code-Review budget is 0 | broken for sampler's entry gate (finding 4) |
| The re-entered review round reads the PR findings as input | broken (finding 3) |
| A friction note through `internal/friction` is legal: Go-authored notes exist through `NotePath`, and its path is non-clobbering | held |
| D3's file list and commit messages share one base | broken (finding 1) |
| A stored ruling is reused across Drift-Recheck | broken (finding 2) |
| The judgement validation criterion | broken (finding 2) |
| An older judgement without `loosened` still reads (tolerant records) | held |
| A new test's loosening is left to D1 | held; it is a stated bound |
| A trailer covers every committer | partly held (finding 5) |
| D5 `run_settings` override: the board shows `loom` keys in `run_settings` | held |
| The peer conflict with `drift-judge-and-line-range` is named as an open risk | held |

**Tool calls**

- Read the `internal/shedadapters/amendment.go` doc comments: my context lacked how a review input reaches a review round.
- Read the `internal/friction/doc.go` doc comments: my context lacked whether Go code may write a friction note.
- Read the sampler Code-Review and PR-Review rows plus an entry-gate grep: my context lacked sampler's Code-Review gate.
- Read `internal/shedengine/doc.go` lines 120–150: my context lacked what happens when a routed-to row's entry gate is off.
