# A webster stop reaches the parent twice: Merriam asks directly and the driver escalates the same question

`webster-stop-asked-once` · Note · bug, webster, loom

Imported from [issue #875](https://github.com/Knatte18/loomyard/issues/875).

Seen (umbrella-anchor, batch 06, 2026-10-25): a batch Merriam could not recover (recover-batch needs_fresh, accept-audit refused) reached the orch on two channels at once. Merriam sent its question by direct message and held, and when Webster ended, the Go driver stopped the run with the same question as a creel stop question (#312, then again #318). Each resumed Webster started a fresh Merriam session that did not know the parent's earlier answer, so one decision took three rounds of messages.

Expected: one stop, one question. Either Merriam writes its outcome and lets the driver's stop carry the question, never also asking the parent directly, or the driver does not re-ask a question the parent already answered; and a fresh Merriam after a parent answer is told that answer (see #849).