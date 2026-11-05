# Webster refusals: short texts, the right cause and a real way forward

`webster-refusal-texts` · Task · bug, webster, quarry · running · Describe

## From note recover-batch-self-way-forward

Imported from [issue #62](https://github.com/Knatte18/loomyard/issues/62).

What happened: a batch whose cards named create targets in one package (a unit, a plan member set and a template file) failed record-batch with batch_failed and create-not-done blocking findings, because the implementer built the code in a different package to avoid an import cycle. The refusal's way forward was 'lym webster recover-batch 07'. Running recover-batch 07 refused with the same findings and the same way forward 'lym webster recover-batch 07'. The run then halted as blocked with anomaly escalation-to-human.

Why it is abnormal: the way forward points at the verb that just refused, so it is a dead end. When the code legitimately lives somewhere other than the card's create target, no verb in the refusal's guidance lets the operator or Webster amend, retarget or waive the create target. Recovery only repeats the same check against the same stale targets.

What to do: when recover-batch refuses on the same findings as the failed record-batch, name a different way forward (for example amend the card's create targets, or escalate with the exact decision needed). Consider a supported verb to retarget a create target when the implementer relocated the unit.

Provenance: task raddle-recipe; notes loom-halt.md, webster-refusal-record-batch.md, webster-refusal-recover-batch.md.

## From note record-batch-signature-mismatch

Imported from [issue #61](https://github.com/Knatte18/loomyard/issues/61).

What happened: `lym webster record-batch` refused a batch because a card's create target (a test method written with a parameter list) did not resolve. quarry answered not_found with the text 'the unit exists and the member is missing'. The refusal then appended, in one line, the full list of every file quarry searched in the unit (hundreds of paths), and only at the very end said the member is declared in one file with a different parameter list than the target named.

Why abnormal: (1) The member exists; only the parameter signature differs, yet the primary message says it is missing, which misdirects the reader. (2) The searched-file list is unbounded and buries the one actionable fact (the declared signature) at the end of a very long line. (3) The way forward given was only `lym webster recover-batch <n>`, which does not tell the card author that the card's create target needs its signature corrected.

What to do: when the member name resolves but the signature differs, report a distinct 'signature mismatch' result that leads with the declared signature and file. Cap or omit the searched-file list (name the count, or the unit, instead). Name the corrective step (fix the card's target signature) next to the recover-batch hint.

Provenance: task slug knut-herschel-bulkley; notes: webster-refusal-record-batch.md

## From note webster-refusal-file-lists

Imported from [issue #63](https://github.com/Knatte18/loomyard/issues/63).

What happened: the webster refusal for record-batch and recover-batch on a batch with several unresolved create targets printed, per target, the complete list of files quarry searched in the unit plus every package in the repo that declares a member of the same name. The unit file list is identical for every target and is repeated for each one. Common names such as Config or LoadConfig produce a long list of unrelated declaring packages. The result is a multi-kilobyte single-line error, written again into the friction note and the loom halt reason context.

Why it is abnormal: the repeated, unrelated detail buries the actionable part (which targets did not resolve) and inflates every note and agent context that carries the refusal.

What to do: list the searched files once per unit rather than per target, cap the declaring-sites list (for example to the nearest few or those in the same parent directory), and keep the one-line per target finding first.

Provenance: task raddle-recipe; notes webster-refusal-record-batch.md, webster-refusal-recover-batch.md.

