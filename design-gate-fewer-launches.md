# Cortex XDR slows every process start of a gate run on OSL-1046

`gate-fewer-launches` · Note · enhancement, gate

## Seen (operator and hub orch, 2026-10-30)

Cortex XDR runs on OSL-1046 and the operator finds every run slow, gates above all.
`lym gate report --slug pr-review-empty-branch` shows `launch_seconds` of 266 of 639 s wall for `code-burler-fix` and 153 of 516 s for `darn-write`, with a wait for a gate slot of about zero, so the time is in the test runs, and a large part of it in starting processes.
The scanner is not visible to `ps` as a process the orch can name, so the cause is the operator's knowledge, not a measurement.

## Idea

- Ask IT for exclusions of the hub directories under `~/Code/Knatte18/loomyard-LYMHUB`, the Go build and module caches, `/tmp/lym-<uid>`, the installed `lym` binaries and `claude`.
- Measure a cold and a warm gate run before and after, so the gain is known; the measurement runs as a darn task with profile `docs`.
- Cut the launches themselves: fewer test binaries started per gate run, if the gate can link tests together or reuse a built binary.
