```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: checking that the round-3 change set agrees with the sibling decisions, criteria and cross-references it touches.

Findings:

1. MEDIUM. Changed text: the D6 paragraph on the scope-full `unruled` entry halt. It says "the resume of that stop, a tackler's included, halts on entry for the parent instead of comparing", and also "a passing compare then returns Done with no agent".
   Summary: the unchanged D6 sentence "Each resume, a tackler's or the arbiter's included, spawns a fresh agent, so the number of attempts is bounded by ..." contradicts both new behaviours.
   Evidence: D6 (the "Each resume" sentence, about line 173) still claims every resume spawns an agent. Under the change, a resume that hits the entry halt spawns none, and neither does the parent's resume that marks acceptances `ruled` and then compares clean. The twin sentence was not adjusted. It should say each resume that reaches a failing compare spawns a fresh agent.

2. LOW. Changed text: D7's new ruling sentence, "Under scope `full` it also carries its ruling: `unruled` ... `ruled` ...", and D6's new ruling mechanism.
   Summary: D7's "with the attempt it ran in" field no longer has a reader, and the support log's Q11 still describes the narrowed rule that Q20 replaces.
   Evidence: D7 still says "the row's live agent session, with the attempt it ran in". The old D6 test "no acceptance by its own session" was its only use, and the fix removed it. Support log Q11 still reads "Narrowed in review round 1: only the agent session's acceptances in the current attempt end it Stuck". Q20 widens that to any `unruled` acceptance at every entry, but Q11 is not marked superseded, as Q12 is for its own replacement.

3. LOW. Changed text: the new D6 entry halt on an `unruled` acceptance, and D6's unchanged `lyx drift waive` refusal "naming `lyx drift accept` or a fix, each followed by resume, at any other `Drift-Verify` halt".
   Summary: at the new entry halt, waive's refusal names accept or fix, but that halt's own ways forward are resume to stand by the acceptances, or withdraw and resume.
   Evidence: D6 says the entry halt "names the parent's ways forward: `lyx loom resume` to stand by the acceptances ... or `lyx drift withdraw` and resume". The waive refusal text was not given that case, so the two disagree on what to do at that halt.

Tool calls made: none; the context already held the review, the decision record and the support log in their changed form.
