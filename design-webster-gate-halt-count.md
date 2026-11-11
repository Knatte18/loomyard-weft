# loom halt: 'verify gate failed after 4 attempt(s) of 3: none named' reports an off-by-one count and no failure

`webster-gate-halt-count` · Note · bug, webster, gate

Imported from [issue #106](https://github.com/Knatte18/loomyard/issues/106).

## What happened

`lym loom` halted blocked at the Webster row with anomaly `escalation-to-human` and the reason:

`verify gate failed after 4 attempt(s) of 3: none named`

## Why it is abnormal

- The message counts 4 attempts against a limit of 3, which reads as an off-by-one in the retry counter or its wording.
- "none named" gives no failing test, project or gate output, so the operator cannot tell what failed or whether the gate itself errored.
- The halt note offers no way forward and names only the trace file.

## What to do

- Make the attempt count and the limit agree, or state that the limit counts retries after the first attempt.
- Include the failing gate, target and the tail of its output in the halt reason, and say explicitly when the gate produced no failure to name.
- Add a way-forward line, as other halt reasons have.

Provenance: task `trip-history-transfer`; note `loom-halt.md`.
