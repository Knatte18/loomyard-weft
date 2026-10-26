# A webster stop reaches the parent twice: Merriam asks directly and the driver escalates the same question

`webster-stop-asked-once` · Task · bug, webster, loom · done

Imported from [issue #875](https://github.com/Knatte18/loomyard/issues/875).

Seen (umbrella-anchor, batch 06, 2026-10-25): a batch Merriam could not recover (recover-batch needs_fresh, accept-audit refused) reached the orch on two channels at once. Merriam sent its question by direct message and held, and when Webster ended, the Go driver stopped the run with the same question as a creel stop question (#312, then again #318). Each resumed Webster started a fresh Merriam session that did not know the parent's earlier answer, so one decision took three rounds of messages.

## Decided (operator, 2026-10-26)

- One stop, one question: a question that needs an answer reaches the parent once.
- The arbiter tries first: a stop Merriam cannot settle goes to the arbiter, and only a question the arbiter hands on reaches the parent.
- The driver never re-asks a question the parent already answered, and a fresh Merriam after a parent answer is told that answer (see #849).
