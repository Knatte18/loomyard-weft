# webster recover-batch refuses with a way forward that names recover-batch itself

`recover-batch-self-way-forward` · Note · bug, webster

Imported from [issue #62](https://github.com/Knatte18/loomyard/issues/62).

What happened: a batch whose cards named create targets in one package (a unit, a plan member set and a template file) failed record-batch with batch_failed and create-not-done blocking findings, because the implementer built the code in a different package to avoid an import cycle. The refusal's way forward was 'lym webster recover-batch 07'. Running recover-batch 07 refused with the same findings and the same way forward 'lym webster recover-batch 07'. The run then halted as blocked with anomaly escalation-to-human.

Why it is abnormal: the way forward points at the verb that just refused, so it is a dead end. When the code legitimately lives somewhere other than the card's create target, no verb in the refusal's guidance lets the operator or Webster amend, retarget or waive the create target. Recovery only repeats the same check against the same stale targets.

What to do: when recover-batch refuses on the same findings as the failed record-batch, name a different way forward (for example amend the card's create targets, or escalate with the exact decision needed). Consider a supported verb to retarget a create target when the implementer relocated the unit.

Provenance: task raddle-recipe; notes loom-halt.md, webster-refusal-record-batch.md, webster-refusal-recover-batch.md.