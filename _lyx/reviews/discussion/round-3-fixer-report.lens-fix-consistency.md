```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-3 change set agrees with its sibling decisions, criteria and cross-references (fix-consistency lens).

1. **MEDIUM.** The changed text is D4's new pre-run covered refusal ("dobby reads it as covered: never a failing pass…") and D7's "Nor is a pass refused at pre-run for a covering pause: it sends no notice". Its twin in D5 was not updated.
   - Summary: a start that races a new pause still goes down D5's refused-start path, so the run gets a notice and a backoff.
   - Evidence: D5 still says "A first pass that batten refuses before the claim posts one notice per refusal text to the nearest orch, and the start is retried from one probe interval, doubling up to an hour".
   - `refuseCovered` runs first in `battenPreRun`, before the seed and the claim, so a start refused for coverage meets that sentence.
   - D7's second marker ("one per task and refusal text for a refused start") is also not exempted.
   - D4's new sentence exempts only "the three-failures notice nor D7's failing-pass marker".

2. **LOW.** The changed text is D4's `--continue-pass` bound, which gained "and no origin-owed marker of the run exists". Its acceptance criterion was not updated.
   - Summary: the criterion still lists only two refusal conditions.
   - Evidence: the criterion reads "A pass given `--continue-pass` for a run not `running` at Run-Shed, or not named in its dobby's state record, is refused naming `lyx batten run &lt;slug&gt;`."
   - It has no marker case.

3. **LOW.** The changed text is D4's "Dobby runs no pass of a run while a pause entry covers the hub or the run's slug" and the D4 line-143 qualifier. One acceptance criterion and D13 are now stale.
   - Summary: a criterion still claims a pass for every run in flight, and D13 still counts covered runs as in flight.
   - Evidence: the criterion reads "a restarted dobby runs a pass of every run in flight, a run Land already pruned included". D13 defines in flight as "a run dobby passes, `running`, with no cancel marker and not named owned elsewhere".
   - A covered running run meets D13's definition, yet under the change it gets no pass at a restart.
   - A dobby holding only covered runs never backs off, though it passes nothing.

4. **LOW.** The changed text is D4's F3 change: "while the mark stands and no pause entry covers the run's slug, dobby runs a full pass of the run on each probe tick". Two untouched lines disagree with it.
   - Summary: D4's line on a marked run, kept unchanged, makes D13 back off on such a run, so "each probe tick" is not met.
   - Evidence: D4 kept "A run so named is not in flight, for D5's `max_runs` count and for D13's backoff alike", and D13 says "With no run in flight, its timer backs off by doubling up to sixty seconds".
   - A dobby holding only a marked run whose owner pause failed backs off to 60 s ticks.
   - So the per-probe re-judge D4 and its new acceptance sentence promise runs at the backed-off cadence instead.

5. **LOW.** The changed text is D4's F3 change, under which dobby keeps running full passes of a marked run whose owner pause failed. D3 was left stale.
   - Summary: D3 says dobby stops passing a taken-over run, which D4 no longer does in the failed-pause case.
   - Evidence: D3 still reads "A run another machine takes over is found by the origin reconcile of a full pass (D4), after a dobby start or a machine wake, and dobby then stops passing it, as D4 says."
   - D4 now has dobby run a full pass on each probe tick while the owner pause failed.
   - D3's list of full passes also leaves out the passes now owed while an origin-owed marker stands.

No further tool calls made.
