# Gate escalation reports "did not pass after 0 attempts" and blocks the step as escalation-to-human

`gate-zero-attempts-halt` · Note · bug, gate, loom, darn

Imported from [issue #92](https://github.com/Knatte18/loomyard/issues/92).

A step halted as blocked with anomaly `escalation-to-human` and the message `gate did not pass after 0 attempts`, pointing to a gate-findings file and `lym loom resume`.
It happened twice at the same producer step (`Darn-Write`), after the step was recorded `stuck` in the history.
Zero gate attempts means the gate never ran, yet the halt reads as a gate failure, so the operator is sent to a findings file for a gate that did not run.

This is abnormal: either the step went stuck before the verify budget was used and the message should say so, or the attempt counter is not recorded.
Make the halt message state why the step is stuck when no gate attempt ran, and make the attempt count reflect the attempts actually made.

Provenance: task slug `slack-presence-auto`; notes `loom-halt-2.md`, `loom-halt-4.md`.