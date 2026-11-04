# Darn-Write session never received its shuttle prompt

`darn-write-prompt-not-sent` · Note · bug, shuttle, darn, loom

Imported from [issue #42](https://github.com/Knatte18/loomyard/issues/42).

What happened: the Darn-Write session's first message only asked it to load the scribe skills and reply ok. The shuttle prompt carrying the darn writer's instructions (.lym/shuttle/<id>/prompt.md) never followed. The session sat idle until the operator typed a nudge, then found the task through _lym/shed/<slug>/status.json and the decision record, and made the change without the instructions. The operator later pasted the prompt path by hand. Only then did the session learn to check the board, commit and write _lym/landing/summary.md. By then it had already run lym gate test itself, which the prompt forbids.

Why abnormal: the darn writer is meant to start from the shuttle prompt. Without it the session worked outside its role limits and needed manual operator intervention. The loom history recorded Darn-Write as done at 19:31:52Z although the prompt was only delivered by hand.

What to do: find why the shuttle prompt was not delivered after the skill-loading first message (ordering or timing of the prompt send versus the session start hook). Deliver it reliably, or have loom detect an idle session that has not received its prompt and re-send it.

Provenance: task transitional-report-note; notes Darn-Write.md.