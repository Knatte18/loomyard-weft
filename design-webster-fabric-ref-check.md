# webster: fabric-reference check flags read-only commands and blocks recovery

`webster-fabric-ref-check` · Note · bug, webster

Imported from [issue #493](https://github.com/Knatte18/loomyard/issues/493).

`lyx webster record-batch 09` refused: an implementer fork's batch failed on four "fabric-reference violation" findings.
The flagged commands were read-only: `lyx fabric clone --help`, `lyx fabric add --help`, `lyx reed up --help`, `cat _lyx/fabric/origin.json`, `git -C loomyard-weft remote get-url origin` and `git ls-remote --heads`.
`lyx webster recover-batch 09` then refused with `needs_fresh`, because the findings name no path and recovery cannot check them.
The run halted `blocked` (escalation-to-human) at Webster until a binary-change re-step let it continue.

This is abnormal: the check treats any command mentioning the fabric as one that "can rewrite run state", including `--help` invocations and plain reads.
A fork that only reads help text or a config file cannot rewrite anything, so the batch fails and halts the run for no real violation.

Suggested fix: classify fabric-referencing commands, and allow `--help`, `cat`/`ls` of fabric files and read-only git queries (`remote get-url`, `ls-remote`).
Also make `recover-batch` able to accept findings that carry no path when every recorded command is read-only.

Provenance: task `daemon-wakeups`; notes `loom-halt.md`, `webster-refusal-record-batch.md`, `webster-refusal-recover-batch.md`, `driver-restep-2a77897a307a6691.md`.
