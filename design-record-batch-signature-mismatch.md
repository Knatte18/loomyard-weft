# webster record-batch: create-not-done refusal lists every searched file and reports a signature mismatch as a missing member

`record-batch-signature-mismatch` · Note · bug, webster, quarry

Imported from [issue #61](https://github.com/Knatte18/loomyard/issues/61).

What happened: `lym webster record-batch` refused a batch because a card's create target (a test method written with a parameter list) did not resolve. quarry answered not_found with the text 'the unit exists and the member is missing'. The refusal then appended, in one line, the full list of every file quarry searched in the unit (hundreds of paths), and only at the very end said the member is declared in one file with a different parameter list than the target named.

Why abnormal: (1) The member exists; only the parameter signature differs, yet the primary message says it is missing, which misdirects the reader. (2) The searched-file list is unbounded and buries the one actionable fact (the declared signature) at the end of a very long line. (3) The way forward given was only `lym webster recover-batch <n>`, which does not tell the card author that the card's create target needs its signature corrected.

What to do: when the member name resolves but the signature differs, report a distinct 'signature mismatch' result that leads with the declared signature and file. Cap or omit the searched-file list (name the count, or the unit, instead). Name the corrective step (fix the card's target signature) next to the recover-batch hint.

Provenance: task slug knut-herschel-bulkley; notes: webster-refusal-record-batch.md